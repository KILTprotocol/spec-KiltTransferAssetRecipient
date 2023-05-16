[![](.maintain/media/kilt-header.png)](https://kilt.io)

# KiltTransferAssetRecipientV2

### Editors

- **Antonio Antonino** - KILT Protocol [antonio@kilt.io](mailto:antonio@kilt.io)

---

## Abstract

This document defines an extension to the service types supported in the [DID Core W3C spec][did-core-spec] by defining the `KiltTransferAssetRecipientV2` service type.
The goal of the endpoints of this class is to expose a collection (i.e., a list) of addresses to which assets of some class can be sent to.
For more information about the KILT DID method, please visit our [official specification][kilt-did-spec].

## Data Structure

A service endpoint of type `KiltTransferAssetRecipientV2` does not include any additional properties compared to what is defined within the [relative section of the official DID Core spec][did-core-spec-services].
Furthermore, endpoints of such type MUST include at least *one* URI for the `serviceEndpoint` property.
Each of the URIs in `serviceEndpoint`, when dereferenced, MUST return **a JSON object** with a mapping from each type of asset to the list of accounts the DID subject controls for that asset, optionally with a description of each account.
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
      "description": "Council account"
    },
    "4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM" : {
      "description": "Personal account"
    }
  },
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": {
    "15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3": {
      "description": "Personal account"
    }
  },
  "eip:1/slip44:60": {
    "0x6b175474e89094c44da98b954eedeac495271d0f": {},
    "0x8f8221AFBB33998D8584A2B05749BA73C37A938A": {
      "description": "NFT sales"
    }
  }
}
```

Each asset is identified by its [CAIP-19 identifier][caip-19-spec].
The value of the property for each asset MUST be another object with the following structure:

* One key for each `account` that can receive assets of the specified type. The account MUST be encoded according to the rules of the chain on which the asset lives. For example, for Spiritnet accounts, the account is the base58-prefixed encoding of the Spiritnet chain ID + the account public key. For Ethereum accounts, it's the 20-byte HEX representation of the account public key, prefixed with `0x`. Other chains have different encoding rules for accounts, and each chain defines the format and encoding logic for public keys representing accounts on those chains.
* For each account, an object with the following properties:
  * [OPTIONAL] `description`: The user-provided description for the specified account.

The example above shows a `KiltTransferAssetRecipientV2` endpoint indicating other parties that the DID subject can accept transfers of the following four assets:

- *DOT tokens* sent to the address `15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3` on the Polkadot relaychain.
- *KILT Spiritnet tokens* sent to either of the addresses `4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM`, or `4nvZhWv71x8reD9gq7BUGYQQVvTiThnLpTTanyru9XckaeWa` on the KILT Spiritnet parachain.
- *KSM tokens* sent to the address `EJDj2GKnx89HTzUkGW8Rk9RoYUmAJHPM8aacWFp3fi1gYUQ` on the Kusama relaychain.
- *ETH non-fungible tokens* sent to the either of the addresses `0x6b175474e89094c44da98b954eedeac495271d0f`, or `0x8f8221AFBB33998D8584A2B05749BA73C37A938A` on the Ethereum mainnet.

### Object Canonicalization and Hashing

The `id` property of the endpoint MUST be the [multibase][multibase] representation of the Blake2b-256 output calculated from the Base64 encoding of the [canonical representation][rfc8785] of the resource dereferenced by the URIs in `serviceEndpoint`.
The multibase chosen MUST yield values that do not contain invalid characters as per the definition of the `id` property in the DID specification, i.e., the resulting `id` MUST still be a valid URI conforming to [RFC3986][rfc3986].

The retrieved resource MUST be canonicalized before being hashed, according to the [RFC 8785 specification][rfc8785].
This canonicalization step is required to ensure that two semantically-equivalent services do not hash to two different values.

Hence, calling `M` the multibase encoding operation of some data, `H` the Blake2b-256 hashing, `B` the binary representation of some information, and `N` the canonicalization step, the service `id` for a given object `O` is `M(H(B(N(O))))`.
For example, with the object `O` being the example `serviceEndpoint` shown above, and the multibase `M` being `base64urlpad`, the resulting service endpoint looks like the following:

```json
{
  "id": "did:kilt:4pqDzaWi3w7TzYzGnQDyrasK6UnyNnW6JQvWRrq6r8HzNNGy#Uif4uWQYSXeeMLAQPNX2aEJvMEmHGkvEqcL-zZdKkRhM=",
  "type": [
    "KiltTransferAssetRecipientV2"
  ],
  "serviceEndpoint": [
    "https://gist.githubusercontent.com/ntn-x2/375d047e6be61d243b9cf645bc94a436/raw/f41c9f4976e09a29e6bd63f84eabdcd0f6cf2f4d/KiltTransferAssetRecipientV2-example.json"
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
      "description": "Council account"
    },
    "4tMSjvHfWBNQw4tYGvkbRp7BBpwAB6S24LuMDcASYgnGnRTM" : {
      "description": "Personal account"
    }
  },
  "polkadot:91b171bb158e2d3848fa23a9f1c25182/slip44:354": {
    "15BQbTH5bKH63WCXTMPxbmpnWeXKpfuTKbpDkfFLXMPvpxD3": {
      "description": "Personal account"
    }
  },
  "eip:1/slip44:60": {
    "0x6b175474e89094c44da98b954eedeac495271d0f": {},
    "0x8f8221AFBB33998D8584A2B05749BA73C37A938A": {
      "description": "NFT sales"
    }
  }
}
`

function main() {
  const jsonInput = JSON.parse(doc)
  const canonicalJson = canonicalize(jsonInput)
  const buffer = Buffer.from(canonicalJson as any)
  const hash = blake2AsU8a(buffer)
  const encoded = multibase.encode('base64urlpad', hash)
  console.log(Buffer.from(encoded).toString('utf-8'))
}

main()
```

## Security Considerations

The list of addresses where the DID owner wants to receive funds MUST always be under the subject's control even if stored off-chain.
This ensures the authenticity and integrity of the list.
Implementations MUST verify that the list of addresses retrieved from the service URI can be hashed and encoded to the same value as the service `id` after following the canonicalization and hashing steps outlined above.
Failure to verify this condition MUST be treated as an attack either towards the DID subject or the entity willing to initiate the asset transfer, and the operation MUST be aborted.

[did-core-spec]: https://www.w3.org/TR/did-core
[kilt-did-spec]: https://github.com/KILTprotocol/spec-kilt-did
[multibase]: https://github.com/multiformats/multibase#multibase-by-example
[did-core-spec-services]: https://www.w3.org/TR/did-core/#services=
[caip-19-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-19.md
[caip-2-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-2.md
[caip-13-spec]: https://github.com/ChainAgnostic/CAIPs/blob/master/CAIPs/caip-13.md
[rfc3986]: https://www.w3.org/TR/did-core/#bib-rfc3986
