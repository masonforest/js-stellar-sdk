---
title: Examples
---

### Creating a simple payment transaction

js-stellar-sdk exposes the [`TransactionBuilder`](https://github.com/stellar/js-stellar-base/blob/master/src/transaction_builder.js) class from js-stellar-base.  This class allows you to add operations to a transaction via chaining.  You can construct a new `TransactionBuilder`, call `addOperation`, call `addSigner`, and `build()` yourself transaction.  Below are two examples, reflecting the two ways of dealing with the [sequence number](#sequence-number).

```javascript
/**
* In this example, we'll create and a submit a payment.  We'll use a
* locally managed sequence number.
*/

var StellarSdk = require('stellar-sdk')

// create the server connection object
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

// create Account object using locally tracked sequence number
var an_account = new StellarSdk.Account("GCEZWKCA5VLDNRLN3RPRJMRZOX3Z6G5CHCGSNFHEYVXM3XOJMDS674JZ", localSequence);

var transaction = new StellarSdk.TransactionBuilder(an_account)
    .addOperation(StellarSdk.Operation.payment({
      destination: "GASOCNHNNLYFNMDJYQ3XFMI7BYHIOCFW3GJEOWRPEGK2TDPGTG2E5EDW",
      asset: StellarSdk.Asset.native(),
      amount: "20000000"
    }))
    .addSigner(StellarSdk.Keypair.fromSeed(seedString)) // sign the transaction
    .build();

server.submitTransaction(transaction)
  .catch(function (err){
    console.log(err);
  });
```


```javascript
/**
* In this example, we'll create and submit a payment, but we'll read the
* sequence number from the server.
*/
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

server.loadAccount("GCEZWKCA5VLDNRLN3RPRJMRZOX3Z6G5CHCGSNFHEYVXM3XOJMDS674JZ")
    .then(function (account) {
        // build the transaction
        var transaction = new StellarSdk.TransactionBuilder(account)
            // this operation funds the new account with XLM
            .addOperation(StellarSdk.Operation.payment({
                destination: "GASOCNHNNLYFNMDJYQ3XFMI7BYHIOCFW3GJEOWRPEGK2TDPGTG2E5EDW",
                asset: StellarSdk.Asset.native(),
                amount: "20000000"
            }))
            .addSigner(StellarSdk.Keypair.fromSeed(seedString)) // sign the transaction
            .build();
        return server.submitTransaction(transaction);
    })
    .then(function (transactionResult) {
        console.log(transactionResult);
    })
    .catch(function (err) {
        console.error(err);
    });
```

### Loading an account transaction history

Let's say you want to look at an account's transaction history.  You can use the `transactions()` command and pass in the account address to `forAccount` as the resource you're interested in.

```javascript
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

server.transactions()
    .forAccount(accountAddress)
    .call()
    .then(function (page) {
        // page 1
        console.log(page.records);
        return page.next();
    })
    .then(function (page) {
        // page 2
        console.log(page.records);
    })
    .catch(function (err) {
        console.log(err);
    });
```
### Streaming an accounts transaction history

js-stellar-sdk provides streaming support for Horizon endpoints using `EventSource`.  For example, pass a streaming `onmessage` handler to an account's transaction call:

```javascript
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

var streamingMessageHandler = function (message) {
    console.log(message);
};

var es = server.transactions()
    .forAccount(accountAddress)
    .stream({
        onmessage: streamingMessageHandler
    })
```

For more on streaming events, please check out [our guide](https://github.com/stellar/horizon/blob/master/docs/guide/responses.md) and this [guide to server-sent events](https://developer.mozilla.org/en-US/docs/Web/API/Server-sent_events/Using_server-sent_events).

### Creating a multi-signature account

In this example, we will:
* Add a second signer to an account
* Set the account's masterkey weight and threshold levels
* Create a multi signature transaction that sends a payment

#### Add a secondary key to the account
```javascript

var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

server.loadAccount(firstAccountAddress)
    .then(function (account) {
        // build the transaction
        var transaction = new StellarSdk.TransactionBuilder(account)
            .addOperation(StellarSdk.Operation.setOptions({
                signer: {
                    address: secondAccountAddress,
                    weight: 1
                }
            }))
            .addSigner(StellarSdk.Keypair.fromSeed(firstAccountSeedString))
            .build();
        return server.submitTransaction(transaction);
    })
    .then(function (transactionResult) {
        console.log(transactionResult);
    })
    .catch(function (err) {
        console.log(err);
    });
```

#### Set Master key weight and threshold weights
```javascript
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

server.loadAccount(firstAccountAddress)
    .then(function (account) {
        // build the transaction
        var transaction = new StellarSdk.TransactionBuilder(account)
            // this operation funds the new account with XLM
                .addOperation(StellarSdk.Operation.setOptions({
                    masterWeight : 1,
                    lowThreshold: 1,
                    medThreshold: 2,
                    highThreshold: 1
                }))
            .addSigner(StellarSdk.Keypair.fromSeed(firstAccountSeedString))
            .build();
        return server.submitTransaction(transaction);
    })
    .then(function (transactionResult) {
        console.log(transactionResult);
    })
    .catch(function (err) {
        console.log(err);
    });
```

#### Create a multi-sig payment transaction
```javascript
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

server.loadAccount(firstAccountAddress)
    .then(function (account) {
        // build the transaction
        var transaction = new StellarSdk.TransactionBuilder(account)
            // this operation funds the new account with XLM
            .addOperation(StellarSdk.Operation.payment({
                destination: destinationAccount,  // can be any destination account
                asset: StellarSdk.Asset.native(),
                amount: "20000000"
            }))
            .addSigner(StellarSdk.Keypair.fromSeed(firstAccountSeedString))
            .addSigner(StellarSdk.Keypair.fromSeed(secondAccountSeedString))
            .build();
        return server.submitTransaction(transaction);
    })
    .then(function (transactionResult) {
        console.log(transactionResult);
    })
    .catch(function (err) {
        console.error(err);
    });
```
#### Multi-sig RPC

Let's say you are a second signer on an account and don't want to share your keypair with the source account.  Instead,
you want to control your keypair on a remote server or separate process.  If the source account can send you the envelope of the transaction it wants to submit, you can sign the envelope remotely like so:


```javascript
var StellarSdk = require('stellar-sdk')
var server = new StellarSdk.Server({hostname:'horizon-testnet.stellar.org', secure: true, port: 443});

// Let's say this function exists on the remote server
var fakeRPCSigner = function(transactionEnvelope) {
    var RPCKeypair = StellarSdk.Keypair.fromSeed(secondAccountSeedString);
    var transaction = new StellarSdk.Transaction(transactionEnvelope);
    // If needed, the remote server can check the transaction
    // for whatever requirements before signing.  For example,
    // let's check to make sure the transaction is only submitting one operation
    if (transaction.operations.length > 1) {
        throw new Error("Only can sign one payment operation");
    }
    // Now let's check to make sure the operation is a low-enough payment
    var operation = transaction.operations[0];
    if (operation.type != "payment" || operation.amount > 1000000) {
        throw new Error("Payment type not payment or amount too high");
    }
    // All the above is optional.  This function only needs to sign
    transaction.sign({RPCKeypair});
    return transaction.toEnvelope();
};

// Build the transaction on the local server
var transaction = new StellarSdk.TransactionBuilder(rootAccount)
    // this operation funds the new account with XLM
    .addOperation(StellarSdk.Operation.payment({
        destination: destAccountAddress,
        asset: StellarSdk.Asset.native(),
        amount: "20000000"
    }))
    .addSigner(StellarSdk.Keypair.fromSeed(rootAccountSeedString))
    .build();
try {
    var RPCSignedTransactionEnvelope = fakeRPCSigner(transaction.toEnvelope());
    server.submitTransaction(new StellarSdk.Transaction(RPCSignedTransactionEnvelope))
        .catch(function (err) {
            console.log(err);
        });
} catch (err) {
    throw new Error("Unable to authorize transaction");
}

```
