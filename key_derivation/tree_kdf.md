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
