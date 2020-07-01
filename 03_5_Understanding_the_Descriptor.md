# 3.5: Understanding the Descriptor

> :information_source: **NOTE:** This is a draft in progress, so that I can get some feedback from early reviewers. It is not yet ready for learning.

You may have noticed a weird "desc:" field in the `listunspent` command of the previous section. Here's what's all about.

## Know about Transferring Addresses

Most of this course presumes that you're working entirely from a single node where you manage your own wallet, sending and receiving payments with the addresses created by that wallet. However, that's not necessarily how the larger Bitcoin ecosystem works. There, you'd more likely to be moving addresses between wallets and even setting up wallets to watch over funds controlled by different wallets. It's most useful if you're interacting with software _other_ than Bitcoin Core, and really need to lean on this sort of compatibility function, but we'll also use a tiny bit of what we learned here in [§6.3](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/06_3_Sending_an_Automated_Multisig.md).

This functionality used to focus on `xpub` and `xprv`, and those are still supported. 

> :book: ***What is xprv?*** An extended private key. This is the combination of a private key and a chain code. It's a private key that a whole sequence of children private keys can be derived from.

> :book: ***What is xpub?*** An extended public key. This is the combination of a public key and a chain code. It's a public key that a whole sequence of children public keys can be derived from.

The fact that you have a "whole sequence of children ... keys" notes that "xpub" and "xprv" aren't standard keys. They're hierarchical keys that can be used to derive whole families of keys, built on the idea of HD Wallets.

> :book: ***What is an HD Wallet?*** Most modern wallets are actually built on [BIP32: Hierarchical Deterministic Wallets](https://github.com/bitcoin/bips/blob/master/bip-0032.mediawiki). This is a hierarchical design where a single seed can be used to generate a whole sequence of keys. The entire wallet may then be restored from that seed, rather than requiring the restoring of every single private key.

> :book ***What is a Derivation Path?*** When you have hierarchical keys, you need to be able to define individual keys as descendents of a seed. For example `[0]` is the 0th key, `[0/1]` is the first son of the 0th key, `[1/0/1]` is the first grandson of the zeroth son of the 1st key. Some keys also contain a `'` after the number to show they're hardened, which protect them from a specific attack that can be used to derive an `xprv` from an `xpub`. You don't need to worry about the specifics, other than the fact that those `'`s will cause you formatting troubles when working from the command line.

However, xpubs and xprvs proved insufficient when the types of public keys multiplied under the [SegWit expansion](https://github.com/BlockchainCommons/Learning-Bitcoin-from-the-Command-Line/blob/master/04_6_Creating_a_Segwit_Transaction.md), thus the need for "output descriptors".

> :book: ***What is an output descriptor?*** A precise description of how to derive a Bitcoin address from a combination of a function and one or more inputs to that function.

The introduction of functions into descriptors is what makes them powerful, because they can be used to transfer all sorts of addresses, from the Legacy addresses that we're working with now to the Segwit and multisig addresses that we'll meet down the road. The function just matches the particular type of address.

## Capture a Descriptor

Descriptors are visible in several commands such as `listunspent` and `getaddressinfo`:
```
$ bitcoin-cli getaddressinfo ms7ruzvL4atCu77n47dStMb3of6iScS8kZ
{
  "address": "ms7ruzvL4atCu77n47dStMb3of6iScS8kZ",
  "scriptPubKey": "76a9147f437379bcc66c40745edc1891ea6b3830e1975d88ac",
  "ismine": true,
  "solvable": true,
  "desc": "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk",
  "iswatchonly": false,
  "isscript": false,
  "iswitness": false,
  "pubkey": "03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388",
  "iscompressed": true,
  "ischange": false,
  "timestamp": 1592335136,
  "hdkeypath": "m/0'/0'/18'",
  "hdseedid": "fdea8e2630f00d29a9d6ff2af7bf5b358d061078",
  "hdmasterfingerprint": "d6043800",
  "labels": [
    ""
  ]
}
```
Here the descriptor is `pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk`.

## Understand a Descriptor

A descriptor is broken into several parts:
```
function([derivation-path]key)#checksum
```
Here's what that all means:
* **Function.** The function that is used to create an address from that key. In this cases it's `pkh`, which is the standard P2PKH legacy address that you met in [§3.3: Setting Up Your Wallet](03_3_Setting_Up_Your_Wallet.md). Similarly, a P2WSH SegWit address would use `wsh` and a P2WPKH address would use `wpkh`.
* **Derivation Path.** This describes what part of an HD wallet is being exported. In this case it's a seed with the fingerprint `d6043800` and then the 18th child of the 0th child of the 0th child (`0'/0'/18'`) of that seed. There may also be a further description after the key: `function([derivation-path]key/more-derivation)#checksum`
* **Key**. The key or keys that are being transferred. This could be something traditional like an `xpub` or `xprv`, it could just be a public key for an address as in this case, it could be a set of addresses for a multi-signature, or it could be something else. This is the core data: the function explains what to do with it.
* **Checksum**. Descriptors are meant to be human transferrable. This checksum makes sure you got it right.

See [Bitcoin Core's Info on Descriptor Support](https://github.com/bitcoin/bitcoin/blob/master/doc/descriptors.md) for more information.

## Examine a Descriptor

You can look at a descriptor with the `getdescriptorinfo` RPC:
```
$ bitcoin-cli getdescriptorinfo "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk"
{
  "descriptor": "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk",
  "checksum": "4ahsl9pk",
  "isrange": false,
  "issolvable": true,
  "hasprivatekeys": false
}
```
Note that it returns a checksum. If you're ever given a descriptor without a checksum, you can learn it with this command:
```
$ bitcoin-cli getdescriptorinfo "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)"
{
  "descriptor": "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk",
  "checksum": "4ahsl9pk",
  "isrange": false,
  "issolvable": true,
  "hasprivatekeys": false
}
```
Besides giving you the checksum, this command also provides useful information like whether a descriptor contains private keys.

The whole point of a descriptor is being able to derive an address in a regular way. This is done with the `deriveaddresses` RPC.
```
$ bitcoin-cli deriveaddresses "pkh([d6043800/0'/0'/18']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk"
[
  "ms7ruzvL4atCu77n47dStMb3of6iScS8kZ"
]
```

## Import a Desciptor

But, the really important thing about a descriptor is that you can take it to another machine and import it. This is done with the `importmulti` RPC using the `desc` option:
```
remote$ bitcoin-cli importmulti '[{"desc": "pkh([d6043800/0'"'"'/0'"'"'/18'"'"']03efdee34c0009fd175f3b20b5e5a5517fd5d16746f2e635b44617adafeaebc388)#4ahsl9pk", "timestamp": "now", "watchonly": true}]'
[
  {
    "success": true
  }
]
```
First, you'll note our first really ugly use of quotes. Every `'` in the derivation path had to be replaced with `'"'"'`. Just expect to have to do that if you're manipulating a descriptor that contains a derivation path. 

Second, you'll note that we flagged this as `watchonly`. That's because we know that it's a public key, so we can't spend with it. If we'd failed to enter this flag, `importmulti` would helpfully have told us something like: `Some private keys are missing, outputs will be considered watchonly. If this is intentional, specify the watchonly flag.`.

> :book: ***What is a watch-only address?***

Using `getaddressesbylabel`, we can now see that our address has correctly been imported into our remote machine!
```
remote$ bitcoin-cli getaddressesbylabel ""
{
  "ms7ruzvL4atCu77n47dStMb3of6iScS8kZ": {
    "purpose": "receive"
  }
}
```
## Summary: Understanding the Descriptor

Descriptors let you pass public keys and private keys among wallets, but more than that, they allow you to precisely and correctly derive addresses of a lot of different sorts from a standardized description format.

> :fire: ***What is the power of descriptors?*** Descriptors allow you to import and export seeds and keys. As a developer, they also allow you to build up the precise sort of addresses that you're interested in creating. For example, we use it in [FullyNoded 2](https://github.com/BlockchainCommons/FullyNoded-2/blob/master/Docs/How-it-works.md) to generate a multi-sig from three seeds. But generally, when you want to move an address from one device to another, this is how you'll do so.

## What's Next?

Advance through "bitcoin-cli" with [Chapter Four: Sending Bitcoin Transactions](04_0_Sending_Bitcoin_Transactions.md).