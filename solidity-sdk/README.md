---
description: SDK for On-chain developers
---

# ⛓ Solidity SDK

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

Include either interfaces, or the source code, as you need to deploy your own Locksmiths or registries. There are also[ canonical registry deployments](../contract-addresses.md) on some EVM chains today.

## Using Locksmith In Your Smart Contracts

There are a number of ways to interact with Locksmith in your smart contracts. Assuming you are accepting Locksmith keys for token gating, you can use the supplied LocksmithKeyChecker extension which provides the _onlyKeyHolder() and onlyKeyOrRootHolder()_ modifiers.

Below is an example on how you would utilize this.

<pre class="language-solidity"><code class="lang-solidity"><strong>pragma solidity ^0.8.23;
</strong><strong>import {LocksmithKeyChecker} from 'locksmith-core/src/utils/LocksmithKeyChecker.sol';
</strong><strong>
</strong><strong>contract MyApp is LocksmithKeyChecker {
</strong><strong>     address public locksmith;
</strong><strong>     constructor(address _locksmith) {
</strong><strong>         locksmith = _locksmith;
</strong><strong>     }
</strong><strong>     
</strong><strong>     function doThing(uint256 keyId) onlyKeyHolder(locksmith, keyId) {
</strong><strong>        // business logic for holding that key
</strong><strong>     }
</strong><strong>     
</strong><strong>     function enableAdminToo(uint256 keyId) onlyKeyOrRootHolder(locksmith, keyId) {
</strong><strong>        // keyId holders, or root key holders of the same ring can do this     
</strong><strong>     }
</strong><strong>}
</strong></code></pre>

You can also respond to receiving a Locksmith key by implementing ERC1155Holder and overriding _onERC1155Received()_. You can use the Locksmith SDK modifier onlyValidLocksmiths() to ensure that all received ERC1155 tokens are valid Locksmith keys.

```solidity
pragma solidity ^0.8.23;
import "openzeppelin-contracts/contracts/token/ERC1155/utils/ERC1155Holder.sol";
import {LocksmithKeyChecker} from 'locksmith-core/src/utils/LocksmithKeyChecker.sol';

contract MyApp is ERC1155Holder, LocksmithKeyChecker {
     function onERC1155Received(address, address, uint256, uint256, bytes memory)
        public virtual override(ERC1155Holder) onlyValidLocksmiths() returns (bytes4) {
         // you know its a locksmith key now
     }
}
```

In the next sections, we will go over all available Locksmith APIs and advanced integration patterns supported.&#x20;
