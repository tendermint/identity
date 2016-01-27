# Identity

## Motivation

We need a more flexible approach to identity.
There is more than one way to authenticate an individual.
Some people want to use hand-held devices for authentication, each with varying degrees of security.
Some people need to extend trust and capabilities to third parties.
We need to be able to revoke keys.
Our trusted devices and third parties change, but we're still the same individual.

Also, because passwords are terrible, and my mom is never going to use a paper/cold HD wallet.

## Overview

There are auth-consumers, users, and keys.
The auth-consumer could be a website or any service that needs to authenticate a user.
One or more keys are associated with a user.
Access to one or more of these keys is needed to authenticate a user for auth-consumers.

## Components on the blockchain

Accounts, the (hash of) keys associated with accounts, and the attributes and capabilities associated with keys are encoded on the blockchain.

Here's a sample of some data that might exist on the blockchain.

```
+------------+----------+-------------------------+-------------------------+
| Account ID | Key Hash | Attributes              | Capabilities            |
+------------+----------+-------------------------+-------------------------+
| Alice      | AAAAAAAA |                         | revoke                  |
| Joe        | BBBBBBBB | limit:"$100/day" rank:3 |                         |
| Joe        | CCCCCCCC | rank:2 auth:false       | revoke(rank>2)          |
| Joe        | DDDDDDDD | rank:1                  | master                  |
+------------+----------+-------------------------+-------------------------+
```

### Accounts

Individual users may have one or more pseudonymous accounts.  In the example above, `Alice` and `Joe` are monikers, not "real" names.
We're not trying to associate any contact or other personal identifiable information with this user, for the purpose of this spec, although it could be done. (e.g. in another column).
We're also not specifying how one gets an account on a blockchain; it may be through a paid transaction, or it may be through some invitation system.

### Keys

Keys aren't stored directly on the blockchain, but their hashes are.  There could be multiple types of keys.

* Cryptographic public keys such as an Ed25519 public key
  e.g. `(ed25519:EEEEEEEE)`
* Another account
  e.g. `(account:Alice)`
* m-of-n threshold with a list of keys
  e.g. `(2-of-3:(ed25519:EEEEEEEE),(ed25519:FFFFFFFF),(account:Alice))`

Notice that a key can contain more keys, recursively.

### Attributes 

Attributes are just a list of key-value pairs. They're used in conjunction with the capabilities system.
Here, Joe uses a convention of `rank:n` to create differently ranked keys, and to mark one key has having a spend limit.

### Capabilities

Capabilities are predefined "verbs" that can be enacted upon the identity system.

* Revoke: revoke a key conditionally for the given account.
* Master: can do anything for the given account, including key revokation and creation.

In the example above, Joe's key with hash `CCCCCCCC` can be used to revoke the key with hash `BBBBBBBB`, but nothing else.
Additionally, the key with hash `CCCCCCCC` cannot be used to authenticate Joe for any service.  It's a key that can only revoke Joe's key -- maybe a third party service.

## Signing up to a website/service

Now, say Joe to sign up with a service or website. The service can challenge the user to authenticate themself by providing a challenge string.
Assuming that the key for hash `BBBBBBBB` is a simple public key, Joe can then respond with a signature of that challenge string.
The service can then see that the public key matchces with the hash `BBBBBBBB`, and that there are no auth restrictions (like `auth:false`).
Complex keys would work similarly.  TODO: draw inspiration from BitID here.

## Logging on to a website/service

TODO: similar.
