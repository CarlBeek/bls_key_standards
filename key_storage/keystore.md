# Keystore

A keystore is a mechanism for storing private keys. It is a JSON file that encrypts a private key and is the standard for interchanging keys between devices as until a user provides their password, their key is safe.
 A keystore is comprised of modules which specify a cryptographic construction and the corresponding parameters for a specific portion of deriving a secret.

## Definition

The process of decrypting the secret held within a keystore can be broken down into 3 sub-processes: obtaining the decryption key, verifying the password and decrypting the secret. Each process has its own functions which can be selected from as well as parameters required for the function all of which are specified within the keystore file itself.

### Decryption key

The decryption key is an intermediate key which is used both to verify the user-supplied password is correct, as well as for the final secret decryption. This key is simply derived from the password, the `function`, and the `params` specified by the`kdf` module as per the keystore file.

| KDF            | `"function"` | `"params"`                                                                               | `"message"` | Definition                                       |
|----------------|--------------|------------------------------------------------------------------------------------------|-------------|--------------------------------------------------|
| PBKDF2-SHA-256 | `"pbkdf2"`   | <ul><li>`"c"`</li><li>`"dklen"`</li><li>`"prf: "hmac-sha256"`</li><li>`"salt"`</li></ul> |             | [RFC 2898](https://www.ietf.org/rfc/rfc2898.txt) |
| scrypt         | `"scrypt"`   | <ul><li>`"dklen"`</li><li>`"n"`</li><li>`"p"`</li><li>`"r"`</li><li>`"salt"`</li></ul>   |             | [RFC 7914](https://tools.ietf.org/html/rfc7914)  |

### Password verification

The password verification verifies step verifies that the password is correct with respect to the `checksum.message`, `cipher.message`, and `kdf`. This is done by appending the `cipher.message` to the 2nd 16 bytes of the decryption key, obtaining its SHA256 hash and verifying whether it matches the `checksum.message`.

Inputs:

* `decryption_key`, the octet string obtained from decryption key process
* `cipher_message`, the octet string obtained from keystore file from `crypto.cipher.message`
* `checksum_message`, the octet string obtained from keystore file from `crypto.checksum.message`

Outputs:

* `valid_password`, a boolean value indicating whether the password is valid

Definitions:

* `a[0:3]` returns a slice of `a` including octets 0, 1, 2
* `a | b` is the concatenation of `a` with `b`

Procedure:

```text
0. DK_slice = decryption_key[16:32]
1. pre_image = DK_slice | cipher_message
2. checksum = SHA256(pre_image)
3. valid_password = checksum == checksum_message
4. return valid_password
```

| Hash       | `"function"`    | `"params"` | `"message"` | Definition                                      |
|------------|-----------------|------------|-------------|-------------------------------------------------|
| SHA-256    | `"sha256"`      |            |             | [RFC 6234](https://tools.ietf.org/html/rfc6234) |

### Secret decryption

The `cipher.function` encrypts the secret using the decryption key, thus to decrypt it, the decryption key along with the `cipher.function` and `cipher.params` must be used.

| Cipher               | `"function"`    | `"params"`               | `"message"` | Definition                                      |
|----------------------|-----------------|--------------------------|-------------|-------------------------------------------------|
| AES-128 Counter Mode | `"aes-128-ctr"` | <ul><li>`"iv"`</li></ul> |             | [RFC 3686](https://tools.ietf.org/html/rfc3686) |

## Path

The `path` indicates where in the key-tree a key originates from. It is a string defined by the [Tree Path Specification](/key_derivation/tree_path.md), if no path is known or the path is not relevant, the empty string, `""` indicates this. The `path` can specify an arbitrary depth within the tree and the deepest node within the tree indicates the depth of the key stored within this file.

## UUID

The `uuid` provided in the keystore is a randomly generated UUID as specified by [RFC 4122](https://tools.ietf.org/html/rfc4122). It is intended to be used as a 128-bit proxy for referring to a particular set of keys or account. This level of abstraction provides a means of preserving privacy for a secret-key or for referring to keys when they are not decrypted.

## Version

The `version` is set to `0`.

## JSON schema

The keystore, at its core, is constructed with modules which allow for the configuration of the cryptographic constructions used password hashing, password verification and secret decryption. Each module is composed of: `function`, `params`, and `message` which corresponds with which construction is to be used, what the configuration for the construction is, and what the input is.

```json
{
    "$ref": "#/definitions/Keystore",
    "definitions": {
        "Keystore": {
            "type": "object",
            "properties": {
                "crypto": {
                    "type": "object",
                    "properties": {
                        "kdf": {
                            "$ref": "#/definitions/Module"
                        },
                        "checksum": {
                            "$ref": "#/definitions/Module"
                        },
                        "cipher": {
                            "$ref": "#/definitions/Module"
                        }
                    }
                },
                "path": {
                    "type": "string"
                },
                "uuid": {
                    "type": "string",
                    "format": "uuid"
                },
                "version": {
                    "type": "integer"
                }
            },
            "required": [
                "crypto",
                "path",
                "uuid",
                "version"
            ],
            "title": "Keystore"
        },
        "Module": {
            "type": "object",
            "properties": {
                "function": {
                    "type": "string"
                },
                "params": {
                    "type": "object"
                },
                "message": {
                    "type": "string"
                }
            },
            "required": [
                "function",
                "message",
                "params"
            ]
        }
    }
}
```
