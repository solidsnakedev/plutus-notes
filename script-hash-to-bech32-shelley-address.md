---
description: >-
  The following document show how to convert a Script Hash or Validator Hash to
  bech32 Shelley Address on both Testnet and Mainnet
---

# Script Hash to Bech32 Shelley Address

## Cabal repl example

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

## Haskell Code example

```haskell
{-# LANGUAGE OverloadedStrings #-}
{-# LANGUAGE TypeApplications  #-}

module Week03.Deploy
    ( writeJSON
    , writeValidator
    , writeUnit
    , writeVestingValidator
    , validator_hash
    , testnet_id
    , script_hash
    , bech32_addr
    ) where

import           Cardano.Api
import           Cardano.Api.Shelley   (PlutusScript (..))
import           Codec.Serialise       (serialise)
import           Data.Aeson            (encode)
import qualified Data.ByteString.Lazy  as LBS
import qualified Data.ByteString.Short as SBS
import           PlutusTx              (Data (..))
import qualified PlutusTx
--import qualified Ledger

import Ledger
import Ledger.Tx.CardanoAPI

import           Week03.Parameterized

dataToScriptData :: Data -> ScriptData
dataToScriptData (Constr n xs) = ScriptDataConstructor n $ dataToScriptData <$> xs
dataToScriptData (Map xs)      = ScriptDataMap [(dataToScriptData x, dataToScriptData y) | (x, y) <- xs]
dataToScriptData (List xs)     = ScriptDataList $ dataToScriptData <$> xs
dataToScriptData (I n)         = ScriptDataNumber n
dataToScriptData (B bs)        = ScriptDataBytes bs

writeJSON :: PlutusTx.ToData a => FilePath -> a -> IO ()
writeJSON file = LBS.writeFile file . encode . scriptDataToJson ScriptDataJsonDetailedSchema . dataToScriptData . PlutusTx.toData

writeValidator :: FilePath -> Ledger.Validator -> IO (Either (FileError ()) ())
writeValidator file = writeFileTextEnvelope @(PlutusScript PlutusScriptV1) file Nothing . PlutusScriptSerialised . SBS.toShort . LBS.toStrict . serialise . Ledger.unValidatorScript

writeUnit :: IO ()
writeUnit = writeJSON "testnet/unit.json" ()

writeVestingValidator :: IO (Either (FileError ()) ())
writeVestingValidator = writeValidator "testnet/vesting.plutus" $ validator $ VestingParam
    { beneficiary = Ledger.PaymentPubKeyHash "c2ff616e11299d9094ce0a7eb5b7284b705147a822f4ffbd471f971a"
    , deadline    = 1643235300000
    }

-- ValidatorHash is the same as Scripthash from Cardano.Api, the only difference is that it uses a differente data constructor
-- ValidatorHaash is coming from Ledger module
validator_hash :: Maybe ValidatorHash
validator_hash = toValidatorHash $ scriptAddress $ validator $ VestingParam (Ledger.PaymentPubKeyHash "c2ff616e11299d9094ce0a7eb5b7284b705147a822f4ffbd471f971a") 1643235300000

testnet_id = Testnet $ NetworkMagic (fromInteger 1231)

-- toCardanoScriptHash is coming from Ledger.Tx.CardanoAPI
script_hash :: Cardano.Api.ScriptHash
script_hash' = do
    case validator_hash of
        Just x -> do
            case toCardanoScriptHash x of
                Right x -> x
                Left x -> error ""
        Nothing -> error ""



bech32_addr = serialiseToBech32 $ makeShelleyAddress testnet_id (PaymentCredentialByScript script_hash) NoStakeAddress
```
