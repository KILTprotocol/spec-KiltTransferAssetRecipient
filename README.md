[![](.maintain/media/kilt-header.png)](https://kilt.io)

# KiltTransferAssetRecipientV1

<!-- TODO: Update snippets, signatures, hashes and IPFS CID once the final version of the document has been agreed on -->

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)
- **Albrecht Weiche** - KILT Protocol [albrecht@kilt.io](mailto:albrecht@kilt.io)

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltTransferAssetRecipientV1` service type.
The goal of the endpoints of this class is to expose a collection (i.e., a list) of addresses to which assets of some class can be sent to.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data Structure

A service endpoint of type `KiltTransferAssetRecipientV1` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Furthermore, endpoints of such type MUST include at least *one* URI for the `serviceEndpoint` property.
Each of the URIs in `serviceEndpoint`, when dereferenced, MUST return **a JSON containing an object** with a mapping from each type of asset to the list of accounts the DID subject controls for that asset, optionally with a description of each account and a proof of ownership of such an account.
An example of the object described is given below.

```json
{
  "polkadot:b0a8d493285c2df73290dfb7e61f870f/slip44:434": [
    {
      "account": "EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ",
      "description": "Personal account"
    }
  ],
  "polkadot:411f057b9107718c9624d6aa4a3f23c1/slip44:2086": [
    {
      "account": "4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa",
      "description": "Council account",
      "proof": {
        "scheme": "schorr-ristretto-25519",
        "digest": "blake2b-256",
        "signature": "0xae5f4d97dd67d45f8c6cb7e4977b9bdd4ccdd14db341995ba5074bccbe27c004a17bcf4a53e1e6a1eaac135c5f2b492e7d84dbbe4d80c221d3caed915f7b1286"
      }
    },
    {
      "account": "4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM",
      "description": "Personal account"
    }
  ],
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": [
    {
      "account": "15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3",
      "description": "Personal account"
    }
  ]
}
```

Each asset is identified by its [CAIP-19 identifier][caip-19-spec].
The value of the property for each asset MUST be a unique set of objects with the following structure:

* `account`: The account encoded according to the chain rules. For example, for Spiritnet accounts, the account is the base58-prefixed encoding of the Spiritnet chain ID + the account public key. For Ethereum accounts, it's the 20-byte HEX representation of the account public key, prefixed with `0x`. Other chains have different encoding rules for accounts, and each chain defines the format and encoding logic for public keys representing accounts on those chains.
* [OPTIONAL] `description`: The user-provided description for the specified account.
* [OPTIONAL] `proof`: The proof of ownership of the specified account. This field is an object with the following structure:
  * `scheme`: The signing scheme (e.g., the curve) used for the signature. For Polkadot accounts, this can be of type `ristretto-25519` for `sr25519` key types, `curve-25519` for `ed25519` key types, and `secp256k1` for `ecdsa` key types. Ethereum accounts are also of type `secp256k1`.
  * `digest`: The digest scheme (e.g., the hash) used before signing the payload. For Polkadot accounts, this is typically `blake2-256`. For Ethereum accounts, this is `keccak-256`.
  * `signature`: The HEX-encoded signature over a specially-crafted payload that is described in the [section below](#proof-registry).

Because there is no single registry that contains all the definitions of the digital signature and hashing schemes, a list of allowed values is specified in the [section below](#proof-registry).

The example above shows a `KiltTransferAssetRecipientV1` endpoint indicating other parties that the DID subject can accept transfers of the following three assets:

- *DOT tokens* sent to the address `15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3` on the Polkadot relaychain.
- *KILT Spiritnet tokens* sent to either of the addresses `4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM`, or `4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa` (verified) on the KILT Spiritnet parachain.
- *KSM tokens* sent to the address `EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ` on the Kusama relaychain.

### Normalisation and Hashing

The `id` property of the endpoint MUST be the [multibase][multibase] representation of the Blake256 output calculated from the Base64 encoding of the normalised representation of the resource dereferenced by the URIs in `serviceEndpoint`.
The multibase chosen must yield values that do not contain invalid characters as per the definition of the `id` property in the DID specification, i.e., the resulting `id` must still be a valid URI conforming to [RFC3986][rfc3986].

The retrieved resource must be normalised before being hashed.
For the scope of this spec, we consider **normalised** a resource that meets the following criteria:

* The set of asset entries is alphabetically sorted by the asset name. Because each asset is unique, there is no possibility of conflict.
* The set of accounts for each asset entry is alphabetically sorted by the account address. Because each address is unique, there is no possibility of conflict.

This is required to ensure that two semantically-equivalent services do not hash to two different values and are hence considered two distinct ones.

Hence, calling `M` the multibase encoding operation of some data, `H` the Blake256 hashing, and `B` the binary representation of some information, the service `id` for a given object `O` is `M(H(B(O)))`.
For example, with the object `O` being the example `serviceEndpoint` shown above, and the multibase `M` being `base64urlpad`, the resulting service endpoint looks like the following:

```json
{
  "id": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#UHfpCR8mCNP5FvNRjN7rLHm7DA8fm7bqB6Pd4fGjaJ4Y=",
  "type": [
    "KiltTransferAssetRecipientV1"
  ],
  "serviceEndpoint": [
    "https://ipfs.io/ipfs/QmYbmJiuUtuNjFA7L7HVCgCCK2GFbxZ2VttFgTvurrfZff"
  ]
}
```

### Proof Registry

As there is no uniform place collecting definitions of hashing and digital signature schemes that are used in the crypto interactions, here is a list, meant to be extended, of all hashing and digital signature schemes that can be found in the `proof` object of a linked account, as well as the definition of what is the payload to generate such proof.

#### Digital Signature Schemes

* `schorr-ristretto-25519`: refers to the Schnorr signature on Ristretto-compressed Ed25519 points. They are typically identified as `sr25519` in the Polkadot ecosystem.
* `eddsa-curve-25519`: refers to the EdDSA digital signature scheme based on Ed25519 points. They are typically identified as `ed25519` in the Polkadot ecosystem.
* `ecdsa-secp256k1`: refers to the EDDSA digital signature scheme based on the secp256k1 curve. They are typically identified as `ecdsa` in the Polkadot ecosystem and are the default scheme for Ethereum and Bitcoin accounts.

#### Hashing Schemes

* `blake2b-256`: Blake2b hash with 256 bit output. They are the most common choice for Polkadot networks.
* `keccak-256`: Hashing scheme used by Ethereum with 256 bit output.

### Signature Generation and Verification

Wherever possible, accounts exposed as part of this spec should also include a `proof` object that provides evidence of control of such an account by the DID subject.
The payload to be signed is the same regardless of the account being exposed, and is the following UTF8-encoded string:

```
Account linked to the DID <DID> and web3name <WEB3NAME>
```

where `<DID>` is a KILT DID, e.g., `did:kilt:4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM` and `<WEB3NAME>` is the web3name linked to the DID, e.g., `w3n:ntn_x2`.

Compliant consumers of this information MUST verify that the DID **and** web3name included in the signed payload match the DID and the web3name of the party expected to receive the asset at the end of the transfer.

The use of both the DID and the web3name as part of the payload to be signed prevents a series of attacks where the same signature could be re-used in a different context, for instance if the web3name is released by its original owner and claimed back by a different entity.
With this proof attached to the address, it is not possible to simply re-use the same address for the new DID, since the payload and hence the signature would not match.

The `signature` field is the HEX-encoded signature over the payload above, after it has been properly prepared for signature following the chain-specific signing conventions.

## Security Considerations

The list of addresses where the DID owner wants to receive funds must always be under the subject's control even if stored off-chain.
This ensures the authenticity and integrity of the list.
Implementations must verify that the list of addresses retrieved from the service URI can be hashed and encoded to the same value as the service `id`.
Failure to verify this condition MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation must be aborted.
Where present, a `proof` must be verified as being generated by the account to which it refers, over the payload described in the [section above](#proof-registry). Failure to verify this proof, e.g., if it belongs to a different DID, a different web3name, or if it simply invalid, MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation must be aborted.

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: https://github.com/KILTprotocol/spec-kilt-did
[multibase]: https://github.com/multiformats/multibase#multibase-by-example
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-13-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-13.md
[rfc3986]: https://www.w3.org/TR/did-core/#bib-rfc3986
