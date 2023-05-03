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
  "polkadot:b0a8d493285c2df73290dfb7e61f870f/slip44:434": {
    "EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ": {
      "description": "Personal account"
    }
  },
  "polkadot:411f057b9107718c9624d6aa4a3f23c1/slip44:2086": {
    "4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa": {
      "description": "Council account",
      "proof": {
        "scheme": "schorr-ristretto-25519",
        "digest": "blake2b-256",
        "signature": "0xae5f4d97dd67d45f8c6cb7e4977b9bdd4ccdd14db341995ba5074bccbe27c004a17bcf4a53e1e6a1eaac135c5f2b492e7d84dbbe4d80c221d3caed915f7b1286"
      }
    },
    "4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM" : {
      "description": "Personal account"
    }
  },
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": {
    "15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3": {
      "description": "Personal account"
    }
  }
}
```

Each asset is identified by its [CAIP-19 identifier][caip-19-spec].
The value of the property for each asset MUST be another object with the following structure:

* One key for each `account` that can receive assets of the specified type. The account MUST be encoded according to the rules of the chain on which the asset lives. For example, for Spiritnet accounts, the account is the base58-prefixed encoding of the Spiritnet chain ID + the account public key. For Ethereum accounts, it's the 20-byte HEX representation of the account public key, prefixed with `0x`. Other chains have different encoding rules for accounts, and each chain defines the format and encoding logic for public keys representing accounts on those chains.
* For each account, an object with the following properties:
  * [OPTIONAL] `description`: The user-provided description for the specified account.
  * [OPTIONAL] `proof`: The proof of ownership of the specified account. This field is an object with the following structure:
    * `scheme`: The signing scheme (e.g., the curve) used for the signature. See the [section below](#digital-signature-schemes) for more details about this field.
    * `digest`: The digest scheme (e.g., the hash) used before signing the payload. For Polkadot accounts, this is typically `blake2b-256`. For Ethereum accounts, this is `keccak-256`. See the [section below](#hashing-schemes) for more details about this field.
    * `signature`: The HEX-encoded signature over a specially-crafted payload that is described in the [section below](#signature-generation-and-verification).

The example above shows a `KiltTransferAssetRecipientV1` endpoint indicating other parties that the DID subject can accept transfers of the following three assets:

- *DOT tokens* sent to the address `15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3` on the Polkadot relaychain.
- *KILT Spiritnet tokens* sent to either of the addresses `4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM`, or `4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa` (verified) on the KILT Spiritnet parachain.
- *KSM tokens* sent to the address `EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ` on the Kusama relaychain.

### Object Canonicalization and Hashing

The `id` property of the endpoint MUST be the [multibase][multibase] representation of the Blake2b-256 output calculated from the Base64 encoding of the [canonical representation][rfc8785] of the resource dereferenced by the URIs in `serviceEndpoint`.
The multibase chosen MUST yield values that do not contain invalid characters as per the definition of the `id` property in the DID specification, i.e., the resulting `id` MUST still be a valid URI conforming to [RFC3986][rfc3986].

The retrieved resource MUST be canonicalized before being hashed, according to the [RFC 8785 specification][rfc8785].
This canonicalization step is required to ensure that two semantically-equivalent services do not hash to two different values.

Hence, calling `M` the multibase encoding operation of some data, `H` the Blake2b-256 hashing, `B` the binary representation of some information, and `N` the canonicalization step, the service `id` for a given object `O` is `M(H(B(N(O))))`.
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

A Typescript snippet showing how to derive the service ID for the example document above is shown below:

```ts
import { blake2AsU8a } from '@polkadot/util-crypto'
import * as multibase from 'multibase'
import canonicalize from 'canonicalize'

const doc = `
{
  "polkadot:b0a8d493285c2df73290dfb7e61f870f/slip44:434": {
    "EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ": {
      "description": "Personal account"
    }
  },
  "polkadot:411f057b9107718c9624d6aa4a3f23c1/slip44:2086": {
    "4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa": {
      "description": "Council account",
      "proof": {
        "scheme": "schorr-ristretto-25519",
        "digest": "blake2b-256",
        "signature": "0xae5f4d97dd67d45f8c6cb7e4977b9bdd4ccdd14db341995ba5074bccbe27c004a17bcf4a53e1e6a1eaac135c5f2b492e7d84dbbe4d80c221d3caed915f7b1286"
      }
    },
    "4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM" : {
      "description": "Personal account"
    }
  },
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": {
    "15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3": {
      "description": "Personal account"
    }
  }
}
`

async function main() {
  const jsonInput = JSON.parse(doc)
  const canonicalJson = canonicalize(jsonInput)
  const buffer = Buffer.from(canonicalJson as any)
  const hash = blake2AsU8a(buffer)
  const encoded = multibase.encode('base64urlpad', hash)
  console.log(Buffer.from(encoded).toString('utf-8'))
}

main()
```

### Proof Registry

As there is no extensive collection for definitions of hashing and digital signature schemes that are used in the crypto interactions, here is a list, meant to be extended, of all hashing and digital signature schemes that can be found in the `proof` object of a linked account, as well as the definition of what is the payload to generate such proof.

#### Digital Signature Schemes

* `schorr-ristretto-25519`: the Schnorr signature on Ristretto-compressed Ed25519 points. They are typically identified as `sr25519` in the Polkadot ecosystem.
* `eddsa-25519`: the EdDSA digital signature scheme based on Ed25519 points. They are typically identified as `ed25519` in the Polkadot ecosystem.
* `ecdsa-secp256k1`: the EDDSA digital signature scheme based on the secp256k1 curve. They are typically identified as `ecdsa` in the Polkadot ecosystem and are the default scheme for Ethereum and Bitcoin accounts.

#### Hashing Schemes

* `blake2b-256`: Blake2b hash with 256 bit output. They are the most common choice for Polkadot networks.
* `keccak-256`: Hashing scheme used by Ethereum with 256 bit output.

### Signature Generation and Verification

Wherever possible, accounts exposed as part of this spec SHOULD include the optional `proof` property that provides evidence of control of such an account by the DID subject.
The payload to be signed is the same regardless of the account being exposed, and is the following UTF8-encoded string:

```
Account linked to the DID <DID>
```

where `<DID>` is a KILT DID, e.g., `did:kilt:4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM`.

The `signature` field is the HEX-encoded signature over the payload above, after it has been properly prepared for signature following the chain-specific message signing conventions.
For example, in the case of Polkadot chains the payload is wrapped in the `<Bytes></Bytes>` tag before being hashed and signed.
For Ethereum networks, the payload is prefixed with the concatenation of `\x19Ethereum Signed Message:\n` and the message length in bytes.

Note that the signature does not include the information about the web3name of the DID subject, meaning that if the DID subject decides to claim a different web3name, the information inside this endpoint would still remain valid.
At the same time, if a different DID subject claims the old web3name, the information becomes invalid as the information about the DID in the signed payload differs from the current DID aliased by the web3name.
In short, the signatures represent proof that, at some point in the past, the DID in the payload was controlling the exposed account.

## Security Considerations

The list of addresses where the DID owner wants to receive funds MUST always be under the subject's control even if stored off-chain.
This ensures the authenticity and integrity of the list.
Implementations MUST verify that the list of addresses retrieved from the service URI can be hashed and encoded to the same value as the service `id` after following the canonicalization and hashing steps outlined above.
Failure to verify this condition MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation MUST be aborted.
Where present, a `proof` MUST be verified as being generated by the account to which it refers, over the payload described [above](#proof-registry).
If the `proof` contains a `scheme` or a `digest` that is not defined in this specification, then the `proof` MUST fail to be verified.
A consumer CAN decide whether the treat the associated account as invalid, or simply as unverified.
Failure to verify this proof, e.g., if it belongs to a different DID or if it simply invalid, MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation MUST be aborted.
The inclusion of the DID as part of the payload to be signed prevents a series of attacks where the same signature could be re-used in a different context, for instance if the DID subject switches to a different web3name and someone else claims the old web3name.
With this proof attached to the address, it is not possible to simply re-use the same address for the new DID, since the payload and hence the signature would not match.

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: https://github.com/KILTprotocol/spec-kilt-did
[multibase]: https://github.com/multiformats/multibase#multibase-by-example
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-13-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-13.md
[rfc3986]: https://www.w3.org/TR/did-core/#bib-rfc3986
[rfc8785]: https://datatracker.ietf.org/doc/html/rfc8785