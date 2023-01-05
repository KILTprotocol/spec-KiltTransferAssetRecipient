[![](.maintain/media/kilt-header.png)](https://kilt.io)

# KiltTransferAssetRecipientV1

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)
- **Albrecht Weiche** - KILT Protocol [albrecht@kilt.io](mailto:albrecht@kilt.io)

### Version History

<!-- TODO: Update before releasing -->
- **v1.0 - Jan.04 2023**: Initial spec publishing

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltTransferAssetRecipientV1` service type.
The goal of the endpoints of this class is to expose a collection (i.e., a list) of addresses to which assets of some class can be sent to.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data structure

A service endpoint of type `KiltTransferAssetRecipientV1` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Furthermore, endpoints of such type MUST include at least *one* URI for the `serviceEndpoint` property.
The `id` property of the endpoint MUST be the [multibase][multibase] representation of the Blake256 output calculated from the Base64 encoding of the resource dereferenced by the URIs in `serviceEndpoint`.
The multibase chosen must yield values that do not contain invalid characters as per the definition of the `id` property in the DID specification, i.e., the resulting `id` must still be a valid URI conforming to [RFC3986][rfc3986].

Hence, calling `M` the multibase encoding operation of some data, `H` the Blake256 hashing, and `E` the Base64Url encoding, the service `id` for a given object `O` is `M(H(E(O)))`.

For example, with an object `O` being the string `Hello, world!`, and the multibase `M` being `base64urlpad`, the resulting service endpoint looks like the following:

```json
{
  "id": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#UtdpEHP5yrgQu9NKxd0KQf2dd5NpXRi1MNgnC4u11WXA",
  "type": [
    "KiltTransferAssetRecipientV1"
  ],
  "serviceEndpoint": [
    "https://ipfs.io/ipfs/QmNUAwg7JPK9nnuZiUri5nDaqLHqUFtNoZYtfD22Q6w3c8"
  ]
}
```

Each of the URIs in `serviceEndpoint`, when dereferenced, MUST return **an object** which contains a mapping from each type of asset to the list of accounts the DID subject controls for that asset.
An example of the object described is given below.

```json
{
  "polkadot:411f057b9107718c9624d6aa4a3f23c1/slip44:2086": [
    "~4qBSZdEoUxPVnUqbX8fjXovgtQXcHK7ZvSf56527XcDZUukq",
    "~4oHvgA54py7SWFPpBCoubAajYrxj6xyc8yzHiAVryeAq574G"
    "~4taHgf8x9U5b8oJaiYoNEh61jaHpKs9caUdattxfBRkJMHvm"
  ],
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": [
    "~15qomv8YFTpHrbiJKicP4oXfxRDyG4XEHZH7jdfJScnw2xnV",
    "~15JGaHWAu1nAEtMMjKZSeu8VsYoTBMmoJq6uwxsDBqmwytSN",
  ]
}
```

Each asset is identified by its [CAIP-19 identifier][caip-19-spec]. The value of the property for each asset MUST be a set of accounts that can take one of two possible representations:

- If the account lives on the same chain as the asset, the [CAIP-2 identifier][caip-2-spec] of the chain is omitted and replaced with a `~` sign.
- If the account lives on a different chain (e.g., when the transfer can happen via some bridges), the full [CAIP-10 identifier][caip-10-spec] for the account must be included.

Hence, the example above shows a `KiltTransferAssetRecipientV1` endpoint indicating other parties that the DID subject can accept transfers of the following two assets:

- *KILT Spiritnet tokens* sent to either of the addresses `4qBSZdEoUxPVnUqbX8fjXovgtQXcHK7ZvSf56527XcDZUukq`, `4oHvgA54py7SWFPpBCoubAajYrxj6xyc8yzHiAVryeAq574G`, or `4taHgf8x9U5b8oJaiYoNEh61jaHpKs9caUdattxfBRkJMHvm` on the Spiritnet parachain.
- *Polkadot tokens* sent to either of the addresses `15qomv8YFTpHrbiJKicP4oXfxRDyG4XEHZH7jdfJScnw2xnV`, or `15JGaHWAu1nAEtMMjKZSeu8VsYoTBMmoJq6uwxsDBqmwytSN` on the Polkadot chain.

## Security considerations

The list of addresses where the DID owner wants to receive funds must always be under the subject's control even if stored off-chain.
This ensures the authenticity and integrity of the list.
Implementations must verify that the list of addresses retrieved from the service URI can be hashed and encoded to the same value as the service `id`.
Failure to verify this condition MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation must be aborted.

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: https://github.com/KILTprotocol/spec-kilt-did
[multibase]: https://github.com/multiformats/multibase#multibase-by-example
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-10-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-10.md
[rfc3986]: https://www.w3.org/TR/did-core/#bib-rfc3986