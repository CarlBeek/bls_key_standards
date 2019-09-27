# Mnemonic

The mnemonic acts as a backup mechanism for a set of keys. Entropy is encoded in human-readable form using words from a fixed vocabulary which are combined into a *mnemonic*. This mnemonic can then be easily backed up by a person via whatever means are appropriate for the security level.

[BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) is a widely adopted standard for such a purpose with robust implementations in multiple programming languages supporting vocabularies in several languages. [BIP39](https://github.com/bitcoin/bips/blob/master/bip-0039.mediawiki) is therefore adopted as the standard for generating a mnemonic and converting the mnemonic to a seed which forms the basis for the rest of this specification.
