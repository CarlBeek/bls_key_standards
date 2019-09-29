# Tree Path

The *path* specifies which key from the key-tree to utilise for what purpose. It achieves this by specifying which node to choose at every depth of the tree. This document should be interpreted in tandem with the [Tree KDF specification](./tree_kdf.md) as that specification describes how to build out a tree of keys and this one, how to traverse that tree and which keys to use for what.

## Specification

The path is a string comprised of the characters ` `, `m`, `/`, and `0` through `9`.

### Path

```text
m / purpose / coin_type /  account / other
```

#### Notation

The notation used within the path is specified within the [Tree KDF specification](./tree_kdf.md), but is summarized again below for convenience.

* `m` Denotes the master node (or root) of the tree
* `i` where `0 <= i < 2^256` is an integer, indicates the `i`th node is used at this level
* `/` Separates the tree into depths, thus `i / j` signifies that `j` is a child of `i`

### Purpose

The purpose is set to 12381 which is the name of the new curve (BLS12-381). Use of purpose `12381` indicates compliance both with this specification along with [the Tree KDF specification](./tree_kdf.md), and [the Mnemonic specification](./mnemonic.md).

### Coin Type

The `coin_type` provides differentiation between different coins thereby allowing a single master-node to be used for many coins while preserving key separation.

### Account

`account` is a field that provides the ability for a user to have distinct sets of keys for different purposes, thereby allowing users to separate their keys and their use into domains.

### Other

The `other` depth is designed to provide a set of related keys that can be used for any purpose. It is required to support this level in the tree, although, for many purposes it will remain `0`.
