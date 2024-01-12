---
description: >-
  The purpose of this knowledge base is to provide developer understanding and
  documentation for the Locksmith SDK and associated solidity and javascript
  client libraries.
---

# Locksmith SDK Documentation

## What is Locksmith?

Locksmith is an open-source on-chain permission primitive designed to provide security composability across smart contracts, applications, and ecosystems.

The Locksmith SDK can be used by dApp and wallet developers alike to add fully decentralized, abstracted account permissions that are fully interoperable across on-chain applications.

## Why use Locksmith?

Today's developer ecosystem has a fragmented security model. This leads to integration challenges, security compromises, and centralization.&#x20;

Today's landscape adds complexity in the following ways:

* **Account Models:** EOAs, multi-sigs, MPC wallets, ERC-4337 smart accounts, and even EVM-compatible chains with native account abstraction all have different means to authorize users and infer delegation.&#x20;
* **Application Support:** Applications can't easily receive delegation from these accounts or verify signatures directly in all cases. Most applications are currently optimized for interacting with EOAs, and require different integration efforts for paymasters, 4337 wallets, etc.
* **Security Fragmentation:** Owning multiple wallets, using bots, or owning smart contracts as developers has no unified security model, ownership, or management across components.

Locksmith solves these complexities by offering secure delegation of ERC1155 tokens that operate as on-chain permissions via token ownership and gating. This quickly enables the following use cases:

* **Extensible Wallets, Delegated Recovery**: As demonstrated  on the Locksmith Wallet Alpha (https://alpha.locksmithwallet.com), you can create fully integrated, multi-vault, multi-user accounts with permissionless and trustless recovery options using the Locksmith NFTs.
* **Safe atomic swaps:** NFTs can encode purpose bound permissions that can be delegated to other programs to utilize. This enables DeFi swaps that do not require token approvals, and can leverage the power of smart accounts to validate balances at the end of the transaction.
* **Soulbound Tokens:** Permissions can be delegated, loaned, and soulbound. Making them perfect abstracted permission markers for any relevant on-chain actor.

## How Does It Work?

The Locksmith SDK at its core is a single ERC1155 contract that contains a secure registry of what are called **Key Rings.**

A Key Ring is a set of associated ERC1155 NFTs (known as Locksmith keys) that are controlled by the ring's **Root Key.** Each key on a ring act as both a permission (token id), and access control list (token holders).

Key rings, and their root keys, can be created by anyone on-chain using the Locksmith contract.

### Root Keys

<figure><img src=".gitbook/assets/DALL_E_2024-01-12_10.29.11_-_A_16-bit_style_master_key_inspired_by_classic_fantasy_video_games._The_key_should_have_a_detailed_and_intricate_design__reminiscent_of_the_pixel_art_s-removebg-preview.png" alt="" width="250"><figcaption><p>Root Keys have admin permissions.</p></figcaption></figure>

When a user creates their new Key Ring, they will be minted a Root Key. This root key gives them full administrative rights over all keys on their ring. Root key holders can:

* **Create Keys:** Root key holders can create new keys to add to their key ring. Rings have no key limit.
* **Copy Keys:** Like physical keys, a root key holder can copy any key on the key ring on demand, including the Root Key.
* **Bind Keys:** Root Key holders can disable transfers of any key on their ring with any address. This enables the Root Key holder to delegate keys to recipients and ensure the key isn't moved or stolen.
* **Burn Keys:** Root key holders can remove any key on their ring from the account of any current holder of that key. Root key holders have complete authority of where all of the keys within their ring are at all times.

### Ring Keys

<figure><img src=".gitbook/assets/DALL_E_2024-01-12_10.32.57_-_A_16-bit_style_silver_key_inspired_by_classic_fantasy_video_games._The_key_should_have_a_detailed_and_intricate_design__reminiscent_of_the_pixel_art_s-removebg-preview (1).png" alt="" width="125"><figcaption><p>Ring Keys denote a delegate permission on a Key Ring.</p></figcaption></figure>

Keys that are created by root keys are called **Ring Keys.** These keys denote an abstract authority that is delegated to the holder by the root key. The value of the permissions assigned to this key rely  in the access that is granted by various applications to the holder.

Ring keys do not offer the ability to create, copy, burn, or soulbind keys like the root key. If a ring key is **soulbound** to its user, the user is not able to transfer, sign away, or otherwise move the key from their account. Only the root key holder can burn or unbind keys.
