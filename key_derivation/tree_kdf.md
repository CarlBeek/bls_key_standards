# Tree Key Derivation Function

The tree KDF serves two purposes: building a tree of related keys, and providing a providing a post-quantum backup signature scheme in the event that BLS12-381 is no longer deemed secure. The tree of keys is useful for building out families of keys from a single source of entropy allowing keys to be derived from one another in a useful pattern while allowing keys to be separated by purpose. The post-quantum backup mechanism takes the form of a Lamport signature and is designed to be a bridge to a new signature scheme in the event that BLS12-381 is deemed insecure.

The tree key derivation function describes how to take a parent key and a child-leaf index and arrive at a child key. This is very useful as one can start with a single source of entropy and from there build out a practically unlimited number of keys. The specification can be broken into two sub-components: generating the master key, and constructing a child key from its parent. The *master key* is used as the root of the tree and then the tree is built in layers from this root.

## The Tree Structure

The key tree is defined purely through the relationship between a child-node and its ancestors. Starting with the root of the tree, the master key, a child node can be derived by knowing the parent's private key and the index of the child. The tree is broken up into depths which are indicated by `/` and the master node is described as `m`. The first child of the master node is therefore described as `m / 0`.

```text
      [m / 0] - [m / 0 / 0]
     /        \
    /           [m / 0 / 1]
[m] - [m / 1]
    \
     ...
      [m / i]
```

## Specification

Every key generated via the key derivation process derives a child key via a set of intermediate Lamport keys. The idea behind the Lamport keys is to provide a quantum secure backup incase BLS12-381 is no longer deemed secure. At a high level, the key derivation process works by using the parent node's privkey as an entropy source for the Lamport private keys which are then hashed together into a compressed Lamport public key, this public key is then hashed into BLS12-381's private key group.

### Helper functions

#### `IKM_to_lamport_SK`

Inputs:

* `IKM`, a secret octet string
* `index`, the index of the desired child node, an integer `0 <= index < 2^256`

Outputs:

* `lamport_SK`, an array of 255 32-octet stings

Definitions:

* `HKDF-Extract` is as defined in [RFC5869](https://tools.ietf.org/html/rfc5869), instantiated with SHA256
* `HKDF-Expand` is as defined in [RFC5869](https://tools.ietf.org/html/rfc5869), instantiated with SHA256
* `K = 32` is the digest size (in octets) of the hash function (SHA256)
* `L = K * 255` is the HKDF output size (in octets)
* `""` is the empty string
* `bytes_split` is a function takes in an octet string and splits it into `K`-byte chunks which are returned as an array

Procedure:

``` text
0. PRK = HKDF-Extract(index, IKM)
1. OKM = HKDF-Expand(PRK, "" , L)
2. lamport_SK = bytes_split(OKM, K)
3. return lamport_SK
```

#### `parent_SK_to_lamport_PK`

Inputs:

* `parent_SK`, the BLS Secret Key of the parent node
* `index`, the index of the desired child node, an integer `0 <= index < 2^256`

Outputs:

* `lamport_PK`, the compressed lamport PK, a 32 octet string

Definitions:

* `I2OSP` is as defined in [RFC3447](https://www.ietf.org/rfc/rfc3447.txt) (Big endian decoding)
* `flip_bits` is a function that returns the bitwise negation of its input
* `""` is the empty string
* `a | b` is the concatenation of `a` with `b`

Procedure:

```text
0. IKM = I2OSP(parent_SK, 32)
1. lamport_0 = IKM_to_lamport_SK(IKM, index)
2. not_IKM = flip_bits(IKM)
3. lamport_1 = IKM_to_lamport_SK(not_IKM, index)
4. lamport_PK = ""
5. for i = 0 to 255
       lamport_PK = lamport_PK | SHA256(lamport_0[i])
6. for i = 0 to 255
       lamport_PK = lamport_PK | SHA256(lamport_1[i])
7. compressed_PK = SHA256(lamport_PK)
8. return compressed_PK
```

#### `HKDF_mod_r`

Inputs:

* `IKM`, a secret octet string.

Outputs:

* `SK`, the corresponding secret key, an integer 0 <= SK < r.

Definitions:

* `HKDF-Extract` is as defined in RFC5869, instantiated with hash H.
* `HKDF-Expand` is as defined in RFC5869, instantiated with hash H.
* `L` is the integer given by ceil((1.5 * ceil(log2(r))) / 8).
* `"BLS-SIG-KEYGEN-SALT-"` is an ASCII string comprising 20 octets.
* `""` is the empty string.
* `OS2IP` is as defined in [RFC3447](https://www.ietf.org/rfc/rfc3447.txt) (Big endian encoding)

Procedure:

```text
1. PRK = HKDF-Extract("BLS-SIG-KEYGEN-SALT-", IKM)
2. OKM = HKDF-Expand(PRK, "", L)
3. SK = OS2IP(OKM) mod r
4. return SK
```

### `derive_child_SK`

The child key derivation function takes in the parent's private key and the index of the child and returns the child private key.

Inputs:

* `parent_SK`, the secret key of the parent node, a big endian encoded integer
* `index`, the index of the desired child node, an integer `0 <= index < 2^256`

Outputs:

* `child_SK`, the secret key of the child node, a big endian encoded integer

Procedure:

```text
0. lamport_PK = parent_SK_to_lamport_PK(parent_SK, index)
1. SK = HKDF_mod_r(lamport_SK)
2. return SK
```

### `derive_master_SK`

Inputs:

* `seed`, the source entropy for the entire tree, a octet string

Outputs:

* `SK`, the secret key of master node within the tree, a big endian encoded integer

Procedure:

```text
0. intermediate_SK = HKDF_mod_r(seed)
1. SK = derive_child_SK(intermediate_SK, 0)
2. return SK
```
