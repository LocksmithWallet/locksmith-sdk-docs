---
description: This section describes the ABI for the IKeyLocker interface.
---

# 🤝 Key Locker APIs

The Key Locker is a special type of **Locksmith Key Vault** that can safely store Locksmith keys, as well as loan them out. The Key Locker is a single, simple smart contract with the following features.

1. **Deposit and Store Locksmith Keys:** The Key Locker can support holding ERC1155 keys from any contract that holds a valid interface for ILocksmith. Any key holder can send a key to the locker for safe storage.
2. **Root-Only Withdrawal:** Only a message sender that holds the root key for a given deposited key's ring can fully and permanently withdrawal the key. This can be done via calling redeemKeys(), but can also be done via burning the key out of the locker directly.
3. **Per-Transaction Key Loans:** For security, all keys that are held by EOAs should be _soulbound_. However, soulbound tokens prevent delegation of authority. So users that hold a particular key, can _borrow_ a copy of it from the KeyLocker should it be there, as long as the key is returned by the end of the user's transaction.

The last feature above is the most important - soulbound keys can be safely and securely loaned out to other smart contracts in a flash-loan like manner. This enables contracts to act on behalf of a keyholder, enabling _dynamic and composable extension of permissions_. All while maintaining security because the loaned key must be returned, and all origin keys are soulbound.

There is no public interface method for depositing keys, it is done via sending the key to the address. So it would look like this:

```solidity
ILocksmith(locksmithAddress).safeTransferFrom(from, keyLockerAddress, keyId, amount, '');
```

The following functions are provided as part of the IKeyLocker interface. The implementation provided also supports ERC165 interface detection against the IKeyLocker interface ID.

## useKeys

A message sender is assuemd to be calling this method while holding a soulbound version of the key they expect to use. If held, the caller's provided destination and calldata will be used to **send** the key into the destination contract with associated metadata.

It is fully expected that the key will be returned to the locker by the end of the transaction, or the entire transaction will revert. This protects the key from being arbitrarily stolen.

It will also ensure that at the end of the transaction the message sender is still holding the soulbound key they used to activate the locker, as to ensure malicious transactions cannot be used to strip permissions from the user.

It is not explicitly enforced that **other** keys cannot be removed from the caller for composability. When using a root key locker, it is critical to trust the destination contract.

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the key declared.
* **KeyNotReturned():** If the loaned out key was not returned to the locker.
* **InsufficientKeys()**: The message sender is asking for more keys than are available.

```solidity
 /**
  * useKeys
  * 
  * @param locksmith the dependency injected locksmith to use.
  * @param keyId the key ID you want to action
  * @param amount the number of keys to take on loan
  * @param destination the target address to send the key, requiring it be returned
  * @param data the encoded calldata to send along with the key to destination.
  */
function useKeys(
    address locksmith, 
    uint256 keyId,
    uint256 amount, 
    address destination,
    bytes memory data) external;
```

## redeemKeys

If a key is held in the locker for use, only a root key holder can remove it. This process is known as "redemption" as the root key is used to redeem the key out of the contract and deactivate the locker for it.

The reason only the key's root key can remove the key is due to security. It is assumed that an unbound key can be put into the locker by anyone, as only the root key holder can create unbound keys. However, we want to avoid situations where key holders can't sign a transaction that steals the extra key in any way. A properly segmented EOA won't be holding a root key ever, and such using the key locker is safe even against malicious transactions.

This method will revert with:

* **KeyNotHeld():** If the message sender does not hold the root key declared.
* **KeyNotRoot():** If the message sender delcares and holds a key that isn't root.
* **InvalidRing():** The message sender is attempting to redeem a key with the wrong root key.
* **InsufficientKeys()**: The message sender is asking for more keys than are available.

```solidity
 /**
  * redeemKeys
  *
  * @param locksmith the dependency injected locksmith to use
  * @param rootKeyId the root key you are using to redeem
  * @param keyId the key ID to redeem.
  * @param amount the number of keys to fully redeem
  */
function redeemKeys(
    address locksmith,
    uint256 rootKeyId,
    uint256 keyId,
    uint256 amount) external;
```

## Emitted Events
