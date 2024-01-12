---
description: SDK for On-chain developers
---

# Solidity SDK

On-chain EVM developers can use the Locksmith SDK to:

* Use as an internal permission registry for their multi-contract application, including contract ownership.
* Provide secure and custom access control list features to users.
* Directly require Locksmith keys as a login and permissioning system.

The combination of users and smart contracts holding keys in their accounts is endless!

## Getting Started

First, you will need to download the NPM package. Yarn is coming soon.

```
npm install locksmith-core
```

In your contracts, you now have access to the following interfaces and their default implementations:

* **ILocksmith.sol:** This file is the interface for the Locksmith contract. These APIs are focused on Key Ring management. Creating Rings and keys, as well as copying, burning, and binding/unbinding keys happen here. Introspection about the Rings themselves is possible, as well as finding all holders of a specific key without off-chain indexing.
* **IKeyLocker.sol:** Because Locksmith keys cannot be stored like regular NFTs, a **Key Locker** is provided as a way to safely store and loan keys out for delegated transactions. This is primarly used by Ring Key holders with a soulbound key. Soulbound key holders can loan out _copies_ of their key for a _single_ transaction without having to trust a third party to give it back. This enables dynamic delegation without opening up key holders to draining attacks or malicious transactions.

Include either interfaces, or the source code, as you need to deploy your own Locksmiths or registries. There are also[ canonical registry deployments](contract-addresses.md) on some EVM chains today.

## Locksmith APIs

The following functions are provided as part of the ILocksmith interface. The implementation provided also supports ERC165 interface detection against the ILocksmith interface ID.



## Key Locker APIs

The following functions are provided as part of the IKeyLocker interface. The implementation provided also supports ERC165 interface detection against the IKeyLocker interface ID.
