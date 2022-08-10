---
description: >-
  The following document show how to convert a Script Hash or Validator Hash to
  bech32 Shelley Address on both Testnet and Mainnet
---

# Script Hash to Bech32 Shelley Address

```haskell
cabal repl
```

```haskell
:set -XOverloadedStrings
```

```haskell
import Cardano.Api
```

```haskell
serialiseToBech32 $ makeShelleyAddress (Mainnet) (PaymentCredentialByScript "028f302bf95b7d9f694b5f320212b604a54618b82fb2053b0a694fa9") NoStakeAddress
```

```haskell
serialiseToBech32 $ makeShelleyAddress (Testnet $ NetworkMagic (fromInteger 1097911063)) (PaymentCredentialByScript "028f302bf95b7d9f694b5f320212b604a54618b82fb2053b0a694fa9") NoStakeAddress
```
