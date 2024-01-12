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

The following functions are provided as part of the ILocksmith interface. The implementation provided also supports ERC165 interface detection against the ILocksmith interface ID, as well as a complete ERC1155 interface for each key.

### createKeyRing

Calling this function will create a key ring with a name, mint the first root key, and give it to the designated receiver. The ring will only have one key on it - the newly minted root key of the ring. There will at first only be one copy of the root key which is sent to the receiver.

This method will only throw an error if the recipient is not a valid ERC1155Receiver, or the caller has supplied insufficient gas for the operation.

```solidity
/**
 * createKeyRing
 *
 * @param ringName    A string defining the name of the key ring encoded as bytes32.
 * @param rootKeyName A string denoting a human readable name of the root key.
 * @param keyUri      The metadata URI for the new root key.
 * @param recipient   The address to receive the root key for this key ring.
 * @return the key ring ID that was created
 * @return the root key ID that was created
 */
 function createKeyRing(
        bytes32 ringName,
        bytes32 rootKeyName,
        string calldata keyUri,
        address recipient) external returns (uint256, uint256);
```

### createKey

Root Key holders can use this method to generate brand new keys and add them to their key ring, and send it to destination wallets.

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the root key declared.
* **KeyNotRoot():** If the declared root key is not actually a root key.
* **InvalidERC1155Receiver()**: If the recipient is not a valid ERC1155Receiver.

```solidity
/**
  * createKey
  *
  * @param rootKeyId The root key the sender is attempting to operate to create new keys.
  * @param keyName   An alias that you want to give the key.
  * @param keyUri    The metadata URI for the newly created key.
  * @param receiver  address you want to receive the ring key.
  * @param bind      true if you want to bind the key to the receiver.
  * @return the ID of the key that was created
  */
function createKey(
    uint256 rootKeyId,
    bytes32 keyName,
    string calldata keyUri,
    address receiver,
    bool bind) external returns (uint256);
```

### copyKey

A root key holder can call this method if there is a key on their ring they want to copy. The root key holder does not have to hold a key in order to make a copy, but the key needs to exist on their key ring. This allows multiple actors to hold the same permission, share the same benefits, and is the main difference between a Locksmith key being an ERC1155 instead of an ERC721.

This method. can only be invoked by using a proper root key, which is held by the message sender.&#x20;

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the root key declared.
* **KeyNotRoot():** If the declared root key is not actually a root key.
* **InvalidRingKey():** If the key to copy doesn't being to the root key ring.
* **InvalidERC1155Receiver()**: If the recipient is not a valid ERC1155Receiver.

```solidity
 /**
  * copyKey
  *
  * @param rootKeyId root key to be used for this operation.
  * @param keyId     key ID the message sender wishes to copy.
  * @param receiver  addresses of the receivers for the copied key.
  * @param bind      true if you want to bind the key to the receiver.
  */
  function copyKey(
        uint256 rootKeyId,
        uint256 keyId,
        address receiver,
        bool bind) external;
```

### soulbindKey

This method is called by a root key holder to make a key soulbound to a specific wallet. When soulbind a key, it is not required that the current target address holds that key. The amount set ensures that when sending a key of a specific ID out of the account, that the acocunt holds at least the amount that is bound to them. For security reasons, its always safest to soulbind the amount, and then send the key, or do it in the same transaction.

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the root key declared.
* **KeyNotRoot():** If the declared root key is not actually a root key.
* **InvalidRingKey():** If the key to copy doesn't being to the root key ring.

```solidity
/**
 * soulbindKey
 *
 * @param rootKeyId the operator's root key
 * @param keyHolder the address to bind the key to
 * @param keyId     the keyId they want to bind
 * @param amount    the amount of keys to bind to the holder
 */
 function soulbindKey(
    uint256 rootKeyId,
    address keyHolder,
    uint256 keyId,
    uint256 amount) external;
```

### burnKey

The root key holder can call this method if they want to revoke a key from a holder. Even if the keys are soulbound, the root key holder can remove them. The soulbound key requirement will still exist on that account. Without changes, any additional keys of that type that are sent would still be soulbound.

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the root key declared.
* **KeyNotRoot():** If the declared root key is not actually a root key.
* **InvalidRingKey():** If the key to copy doesn't being to the root key ring.

This method will also throw ERC1155 associated errors if you attempt to burn keys from addresses that have an insufficient key balance.

```solidity
/**
 * burnKey
 *
 * @param rootKeyId root key for the associated ring.
 * @param keyId     id of the key you want to burn.
 * @param holder    address of the holder you want to burn from.
 * @param amount    the number of keys you want to burn.
 */
 function burnKey(
    uint256 rootKeyId,
    uint256 keyId,
    address holder,
    uint256 amount) external;
```

### getRingInfo

Given a ring ID, provides associated metadata back to the user. This method can be called by anyone, not just root key holders.

<pre class="language-solidity"><code class="lang-solidity">/**
 * getRingInfo()
 *
<strong> * @param ringId The ID of the ring to inspect.
</strong> * @return The ring ID back as verification.
 * @return The human readable name for the ring.
 * @return The root key ID for the ring.
 * @return The list of keys for the ring.
 */
function getRingInfo(uint256 ringId) external view 
  returns (uint256, bytes32, uint256, uint256[] memory);
</code></pre>

### getKeysForHolder

This method will return an array of IDs of the keys held for the provided address. These keys exist only within the context of this Locksmith, and can be associated across any Key Ring.&#x20;

```solidity
/**
 * getKeysForHolder()
 *
 * @param  holder the address of the key holder you want to see.
 * @return an array of key IDs held by the user.
 */
function getKeysForHolder(address holder) external view returns (uint256[] memory);
```

### getHolders

For a given key ID, this method will return all of the addresses on the chain that hold it.

```solidity
/**
  * getHolders()
  *
  * @param  keyId the key ID to look for.
  * @return an array of addresses that hold that key.
  */
 function getHolders(uint256 keyId) external view returns (address[] memory);
```

### getSoulboundAmount

Returns the number of keys a given holder must maintain when sending the associated key ID out of their address.&#x20;

```solidity
/**
 * getSoulboundAmount
 *
 * @param account   The wallet address you want the binding amount for.
 * @param keyId     The key id you want the soulbound amount for.
 * @return the soulbound token requirement for that wallet and key id.
 */
function getSoulboundAmount(
    address account,
    uint256 keyId) external view returns (uint256);
```

### isRootKey

Returns true if the key is identified as a root key of a key ring, false otherwise.

```solidity
/**
 * isRootKey()
 *
 * @param keyId the key id in question
 * @return true if the key Id is the root key of it's associated key ring
 */
function isRootKey(uint256 keyId) external view returns(bool);
```

### inspectKey

Returns a bunch of associated metadata about a key, given an ID.

```solidity
/**
 * inspectKey()
 *
 * @return true if the key is a valid key
 * @return alias of the key
 * @return the ring id of the key (only if its considered valid)
 * @return true if the key is a root key
 * @return the keys associated with the given ring
 */
function inspectKey(uint256 keyId) public view 
  returns (bool, bytes32, uint256, bool, uint256[] memory);
```

### hasKeyOrRoot

Determines if the given address holds either the key specified, or the ring's root key. This is used by contracts to enable what is called "root key escalation," and prevents the need for root key holders to hold every key to operate as an admin.

```solidity
/**
 * hasKeyOrRoot()
 *
 * @param keyHolder the address of the keyholder to check.
 * @param keyId     the key you want to check they are holding.
 * @return true if keyHolder has either keyId, or the keyId's associated root key.
 */
function hasKeyOrRoot(
    address keyHolder,
    uint256 keyId) external view returns (bool);
```

### validateKeyRing

This method can be used to determine if a set of keys belong to the same ring. This is used as a validation method and will **revert** if the keys do not belong to the same ring. This is extremely useful when taking a set of keys and ensuring each is part of the same ring first.

You can use the _allowRoot_ flag to enable. cases where passing in the root key as part of the set is acceptable. Setting it to false enables you to quickly detect if one of the keys provided is the root key. Remember, this method will revert also based on _allowRoot_ semantics.

This method will revert with:

* **InvalidRing():** The provided ringId doesn't exist in the registry.
* **InvalidRingKeySet():** The keys are not all part of the same ring, or _allowRoot_ semantics were not followed.

```solidity
/**
 * validateKeyRing()
 *
 * @param ringId    the ring ID you want to validate against
 * @param keys      the supposed keys that belong to the ring
 * @param allowRoot true if providing the ring's root key as input is acceptable
 * @return true if valid, or will otherwise revert.
 */
function validateKeyRing(
    uint256 ringId,
    uint256[] calldata keys,
    bool allowRoot) external view returns (bool);
```

### Emitted Events

The following events are defined in the Locksmith interface and are emitted during locksmith operations. All standard ERC1155 events also apply with regards to token activity.

#### keyRingCreated

```solidity
/**
 * keyRingCreated
 *
 * This event is emitted when a key ring is created and a root key is minted.
 *
 * @param operator  the message sender and creator of the key ring.
 * @param ringId    the resulting id of the new key ring.
 * @param ringName  the ring's human readable name, encoded as a bytes32.
 * @param recipient the address of the root key recipient
 */
 event keyRingCreated(
        address operator,
        uint256 ringId,
        bytes32 ringName, 
        address recipient);
```

#### keyMinted

```solidity
/**
 * keyMinted
 *
 * This event is emitted when a key is minted. This event
 * is also emitted when a root key is minted upon ring creation.
 *
 * The operator will always hold the root key for the ringId.
 *
 * @param operator the creator of the ring key.
 * @param ringId   the key ring ID they are creating the key on.
 * @param keyId    the key ID that was minted by the operator.
 * @param receiver the receiving wallet address where the keyId was deposited.
 */
 event keyMinted(
        address operator,
        uint256 ringId, 
        uint256 keyId,
        address receiver);
```

#### keyBurned

```solidity
 /**
  * keyBurned
  *
  * This event is emitted when a key is burned by the root key
  * holder.
  *
  * @param operator the root key holder requesting the burn.
  * @param ringId   the ring ID they are burning from.
  * @param keyId    the key ID that was burned.
  * @param target   the address of the wallet that had keys burned.
  * @param amount   the number of keys burned in the operation.
  */
 event keyBurned(
         address operator,
         uint256 ringId,
         uint256 keyId,
         address target,
         uint256 amount);
```

#### setSoulboundKeyAmount

```solidity
/**
 * setSoulboundKeyAmount
 *
 * This event fires when the state of a soulbind key is set.
 *
 * @param operator  the root key holder that is changing the soulbinding
 * @param keyHolder the address we are changing the binding for
 * @param keyId     the Id we are setting the binding state for
 * @param amount    the number of tokens this address must hold
 */
 event setSoulboundKeyAmount(
        address operator,
        address keyHolder,
        uint256 keyId,
        uint256 amount);
```

## Key Locker APIs

The following functions are provided as part of the IKeyLocker interface. The implementation provided also supports ERC165 interface detection against the IKeyLocker interface ID.
