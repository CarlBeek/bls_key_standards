
# DEPRECATED: Key standards for BLS12-381

__These specifications have been deprecated in favour of EIPs [2333](https://github.com/ethereum/EIPs/pull/2333), [2334](https://github.com/ethereum/EIPs/pull/2334), [2335](https://github.com/ethereum/EIPs/pull/2335).__

In order to further enhance interoperability, it is useful to define standards in addition to the [BLS Signature Scheme](https://tools.ietf.org/html/draft-irtf-cfrg-bls-signature-00), and [BLS hash to curve](https://tools.ietf.org/html/draft-irtf-cfrg-hash-to-curve-04) standards.

These standards are specifically targeted at the blockchain industry, however wherever possible, a non-specific standard is used to ensure the proliferation of the standards.

This set of standards consists of the following documents:

## Key derivation

* [Mnemonic](key_derivation/mnemonic.md) - A human-readable backup for secret keys which is converted into a seed
* [Tree KDF](key_derivation/tree_kdf.md) - Building out a tree of keys from a single seed
* [Tree Path](key_derivation/tree_path.md) - Allocating purposes to each key from the key tree

## Key checksums

* [Public key checksum](key_checksums/public_key_checksum.md) - Checksums for addresses

## Key storage

* [Keystore](key_storage/keystore.md) - Encrypted files for storing and transferring private keys
