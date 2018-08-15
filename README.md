# SwiftyEOS

[![Build Status](https://travis-ci.org/ProChain/SwiftyEOS.svg?branch=master)](https://travis-ci.org/ProChain/SwiftyEOS)

SwiftyEOS is an open-source framework for interacting with [EOS](https://github.com/EOSIO/eos), written in Swift.

## Features

- EOS key pairs generation
- Private key import 
- Signature hash
- Basic RPC API(chain/history) query client
- Transaction(EOS token transfer)
- Helper class for handling offline wallet on iOS
- Encrypt/decrypt importing private key on iOS

## How to use it

1. Copy the `Libraries` and `Sources` folders into your project, without `main.swift`.
2. Delete `Sources/Utils/iOS` if not targeted to iOS platform.
3. Add `Libraries/include` into Header Search Path.
4. Set `Libraries/include/Bridging-Header.h` as Objective-C Bridging Header. If you've got your own bridging header , copy all the `import` content in that file and paste into your own one.
5. Compile and pray.

## Key Pairs Generation

SwiftyEOS now support secp256k1 key pairs.

> There are bugs around secp256r1 key pairs generation but I cannot figure out why. Unit tests created from `cleos create key --r1` wouldn't pass. You may not consider secp256r1 as an option since the `cleos wallet` command cannot import those keys, either.

### Generate a random key pair:

```swift
let (pk, pub) = generateRandomKeyPair(enclave: .Secp256k1)
```

Easy, right?

```swift
print("private key: \(pk!.wif())")
print("public key : \(pub!.wif())")

// private key: PVT_K1_5HxrYTdZX89zodtJhTzCk87MfNZAkiBRfFvSX8kacYjtwaDpTkL
// public key : PUB_K1_4yDYdmcVcXxAxeNsUWRG7x9FKQE4HbJZdzgZFv1AYxk6oSVcLd
```

The `PVT_K1_` and the `PUB_K1_` prefixes are the parts of standard key presentations. But old styles are also supported in EOS system and SwiftyEOS:

```swift
print("private key: \(pk!.rawPrivateKey())")
print("public key : \(pub!.rawPublicKey())")

// private key: 5HxrYTdZX89zodtJhTzCk87MfNZAkiBRfFvSX8kacYjtwaDpTkL
// public key : EOS4yDYdmcVcXxAxeNsUWRG7x9FKQE4HbJZdzgZFv1AYxk6oSVcLd
```

### Import existing key:

```swift
let importedPk = try PrivateKey(keyString: "5HxrYTdZX89zodtJhTzCk87MfNZAkiBRfFvSX8kacYjtwaDpTkL")
let importedPub = PublicKey(privateKey: importedPk!)
```

With delimiter and prefix:

```swift
let importedPk = try PrivateKey(keyString: "PVT_K1_5HxrYTdZX89zodtJhTzCk87MfNZAkiBRfFvSX8kacYjtwaDpTkL")
let importedPub = PublicKey(privateKey: importedPk!)
```

## RPC API

```swift
EOSRPC.sharedInstance.chainInfo { (chainInfo, error) in
    if error == nil {
        print("Success: \(chainInfo!)")
    } else {
        print("Error: \(error!.localizedDescription)")
    }
}
```

Currently we have some basic rpc endpoints, you can find it at `Sources/SwiftyEOS/Network`

## iOS Key Store

We have helpers for iOS offline wallet management at `SEWallet.swift`.

With `SEWallet.swift` you can easily store AES encrypted key info into file system. The default location is app's sandbox Library.

> Multiple wallets management is not supported yet.

### Create new wallet on iOS

In Objective-C:

```objective-c
[SEKeystoreService.sharedInstance newAccountWithPasscode:passcode succeed:^(SELocalAccount *account) {
} failed:^(NSError *error) {
        
}];
```

### Retrieve saved wallet

```objective-c
[SELocalAccount currentAccount];
```

It will return nil if there's no saved wallet.

## Transactions

Transaction behaviors are not **fully** supported yet, but you still can have a try with sample code at `main.swift`.

The related documents will be provided once the whole function is done. 

1. Currency transfer (2018.08.15)

### Transfer Currency

```swift
var transfer = Transfer()
transfer.from = "agoodaccount"
transfer.to = "gq3dinztgage"
transfer.quantity = "1.0000 EOS"
transfer.memo = "eureka"

Currency.transferCurrency(transfer: transfer, privateKey: importedPk!, completion: { (result, error) in
    if error != nil {
        if error is RPCErrorResponse {
            print("\((error as! RPCErrorResponse).errorDescription())")
        } else {
            print("other error: \(String(describing: error?.localizedDescription))")
        }
    } else {
        print("done.")
    }
})
```