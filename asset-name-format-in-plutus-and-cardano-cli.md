---
description: >-
  We're going to see how Asset Name Format is handled by cardano-cli and Plutus
  library "Ledger.Value.CurrencySymbol"
---

# Asset Name Format in Plutus and cardano-cli

According to the Cardano ledger, an asset must be represented in hex format (some other names are: ByteString, base16), it means if asset name is "abcd" its hex format will be "61626364"

## Asset Name Format in cardano-cli

When interacting with an asset in cardano-cli, one may use a bash command called <mark style="color:yellow;">xxd</mark>

```bash
token_name = "abcd"
token_name_hex=$(echo -n ${2} | xxd -ps | tr -d '\n')
echo ${token_name_hex}
61626364
```

## Asset Name Format in plutus

```nix
[nix-shell:~/plutus-pioneer-program/code/week01]$ cabal repl
Prelude > import Data.Text
Prelude > import Ledger.Value 
Prelude > import PlutusTx.Builtins.Internal
Prelude > Ledger.Value.CurrencySymbol (encodeUtf8 $ BuiltinString $ Data.Text.pack "abcd")
61626364
```

### Understanding the plutus functions

This is the data flow of String to ByteString in plutus:

String <mark style="color:green;">-></mark> Text <mark style="color:green;">-></mark> BuiltinString <mark style="color:green;">-></mark> BuiltinByteString <mark style="color:green;">-></mark> CurrencySymbol

### 1. Import Data.Text

```haskell
Prelude > import Data.Text
Prelude > :i Data.Text.pack
pack :: String -> Text  -- Defined in ‘Data.Text’
```

Data.Text.pack function takes a String type and returns a Text type

Text type is not the default for strings hence we need to convert a String type to Text using Data.Text.pack function

### 2. Import PlutusTx.Builtins.Internal

We need the constructor BuiltinString and the function encodeUtf8

```haskell
Prelude > import PlutusTx.Builtins.Internal
```

### BuiltinString constructor

```haskell
Prelude > :i BuiltinString
type BuiltinString :: *
newtype BuiltinString = BuiltinString Text
        -- Defined in ‘PlutusTx.Builtins.Internal’
instance Eq BuiltinString
  -- Defined in ‘PlutusTx.Builtins.Internal’
instance Ord BuiltinString
  -- Defined in ‘PlutusTx.Builtins.Internal’
instance Show BuiltinString
  -- Defined in ‘PlutusTx.Builtins.Internal’
```

We can see that <mark style="color:green;">BuiltinString</mark> is a type constructor `newtype BuiltinString = BuiltinString Text`, therefore we can construct a <mark style="color:green;">BuiltinString</mark> from a <mark style="color:green;">Text</mark>

### encodeUtf8 function

```haskell
Prelude > :i encodeUtf8
encodeUtf8 :: BuiltinString -> BuiltinByteString
        -- Defined in ‘PlutusTx.Builtins.Internal’
```

We now encode the <mark style="color:green;">BuiltinString</mark> type to <mark style="color:green;">BuiltinByteString</mark> type

### 3. Import Ledger.Value

CurrencySymbol is a constructor that takes a <mark style="color:green;">BuiltinByteString</mark> type and returns a <mark style="color:green;">CurrencySymbol</mark> type

```haskell
Prelude > import Ledger.Value
Prelude > :i Ledger.Value.CurrencySymbol 
type CurrencySymbol :: *
newtype CurrencySymbol
  = CurrencySymbol {unCurrencySymbol :: BuiltinByteString}
        -- Defined in ‘plutus-ledger-api-0.1.0.0:Plutus.V1.Ledger.Value’
instance Eq CurrencySymbol
  -- Defined in ‘plutus-ledger-api-0.1.0.0:Plutus.V1.Ledger.Value’
instance Ord CurrencySymbol
  -- Defined in ‘plutus-ledger-api-0.1.0.0:Plutus.V1.Ledger.Value’
instance Show CurrencySymbol
  -- Defined in ‘plutus-ledger-api-0.1.0.0:Plutus.V1.Ledger.Value’
```

### 4. Compose the function

```haskell
Prelude > Ledger.Value.CurrencySymbol (encodeUtf8 $ BuiltinString $ Data.Text.pack "abcd")
61626364
```
