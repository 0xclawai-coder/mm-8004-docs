# MoltMarketplace Contract — Feature Reference

> **Contract**: `MoltMarketplace.sol`
> **Solidity**: `^0.8.28` | **Framework**: Foundry
> **Tests**: 102/102 passing (70 base + 32 ERC-8004 integration)

---

## Constants

| Name | Value | Description |
|------|-------|-------------|
| `MAX_PLATFORM_FEE_BPS` | 1,000 | 10% hard cap on platform fee |
| `BPS_DENOMINATOR` | 10,000 | Basis points denominator |
| `MAX_BUNDLE_SIZE` | 20 | Max NFTs per bundle listing |

---

## 1. Fixed-Price Listings

| Function | Signature | Description |
|----------|-----------|-------------|
| `list` | `list(nftContract, tokenId, paymentToken, price, expiry) → listingId` | Create listing. NFT escrowed. `paymentToken=0x0` for native. |
| `buy` | `buy(listingId) payable` | Buy at listed price. Native: send ETH. ERC-20: approve first. |
| `cancelListing` | `cancelListing(listingId)` | Seller cancels. NFT returned from escrow. |
| `updateListingPrice` | `updateListingPrice(listingId, newPrice)` | Seller updates price without re-listing. |

**Events**: `Listed`, `Bought`, `ListingCancelled`, `ListingPriceUpdated`

---

## 2. Offers

| Function | Signature | Description |
|----------|-----------|-------------|
| `makeOffer` | `makeOffer(nftContract, tokenId, paymentToken, amount, expiry) → offerId` | ERC-20 only. Checks balance + allowance. |
| `acceptOffer` | `acceptOffer(offerId)` | NFT owner accepts. Pulls ERC-20, distributes funds, transfers NFT. |
| `cancelOffer` | `cancelOffer(offerId)` | Offerer cancels. |

**Events**: `OfferMade`, `OfferAccepted`, `OfferCancelled`

---

## 3. Collection Offers

Offer on **any tokenId** from a specific NFT collection.

| Function | Signature | Description |
|----------|-----------|-------------|
| `makeCollectionOffer` | `makeCollectionOffer(nftContract, paymentToken, amount, expiry) → offerId` | No specific tokenId. ERC-20 only. |
| `acceptCollectionOffer` | `acceptCollectionOffer(offerId, tokenId)` | NFT owner chooses which tokenId to sell. |
| `cancelCollectionOffer` | `cancelCollectionOffer(offerId)` | Offerer cancels. |

**Events**: `CollectionOfferMade`, `CollectionOfferAccepted`, `CollectionOfferCancelled`

---

## 4. English Auctions

| Function | Signature | Description |
|----------|-----------|-------------|
| `createAuction` | `createAuction(nftContract, tokenId, paymentToken, startPrice, reservePrice, buyNowPrice, startTime, duration) → auctionId` | NFT escrowed. See params below. |
| `bid` | `bid(auctionId, amount) payable` | Native: `msg.value` used, `amount` ignored. ERC-20: `amount` used. |
| `settleAuction` | `settleAuction(auctionId)` | After `endTime`. Distributes funds or returns NFT. |
| `cancelAuction` | `cancelAuction(auctionId)` | Seller cancels. Only if no bids. |

### Auction Parameters

| Param | Value | Behavior |
|-------|-------|----------|
| `reservePrice` | `0` = disabled | If highest bid < reserve at settle: NFT returned, bidder refunded. |
| `buyNowPrice` | `0` = disabled | If bid >= buyNow: settles immediately at buyNow price. |
| `startTime` | `0` = start now | Future timestamp: bidding blocked until then. |
| `duration` | 1h–30d | Auction length from startTime. |

### Auction Mechanics

- **Min increment**: 5% above current highest bid
- **Anti-snipe**: Bid in last 10 min → extends end by 10 min (emits `AuctionExtended`)
- **Bid count**: Tracked in `Auction.bidCount`
- **Buy-now**: Refunds previous bidder, settles at `buyNowPrice` (emits `AuctionBuyNow`)
- **Reserve not met**: Refunds bidder, returns NFT (emits `AuctionReserveNotMet`)

**Events**: `AuctionCreated`, `BidPlaced`, `AuctionSettled`, `AuctionCancelled`, `AuctionExtended`, `AuctionBuyNow`, `AuctionReserveNotMet`

---

## 5. Dutch Auctions

Price decreases linearly from `startPrice` to `endPrice` over `duration`.

| Function | Signature | Description |
|----------|-----------|-------------|
| `createDutchAuction` | `createDutchAuction(nftContract, tokenId, paymentToken, startPrice, endPrice, duration) → auctionId` | NFT escrowed. Starts immediately. |
| `buyDutchAuction` | `buyDutchAuction(auctionId) payable` | Buy at current price. Excess refunded (native). |
| `cancelDutchAuction` | `cancelDutchAuction(auctionId)` | Seller cancels anytime. |

### Price Formula
```
currentPrice = startPrice - ((startPrice - endPrice) * elapsed / duration)
```
After `endTime`: price = `endPrice`.

**View**: `getDutchAuctionCurrentPrice(auctionId) → uint256`

**Events**: `DutchAuctionCreated`, `DutchAuctionBought`, `DutchAuctionCancelled`

---

## 6. Bundle Listings

Sell multiple NFTs (up to 20) as a single package.

| Function | Signature | Description |
|----------|-----------|-------------|
| `createBundleListing` | `createBundleListing(nftContracts[], tokenIds[], paymentToken, price, expiry) → bundleId` | All NFTs escrowed. Max 20 items. |
| `buyBundle` | `buyBundle(bundleId) payable` | Buy entire bundle. All NFTs transferred. |
| `cancelBundleListing` | `cancelBundleListing(bundleId)` | Seller cancels. All NFTs returned. |

**Note**: Royalty check uses first NFT in the bundle.

**Events**: `BundleListed`, `BundleBought`, `BundleListingCancelled`

---

## 7. Admin Functions

| Function | Signature | Description |
|----------|-----------|-------------|
| `setPlatformFee` | `setPlatformFee(newFeeBps)` | Owner only. Max 10% (1000 bps). |
| `setFeeRecipient` | `setFeeRecipient(newRecipient)` | Owner only. Non-zero address. |
| `transferOwnership` | `transferOwnership(newOwner)` | Owner only. |
| `pause` | `pause()` | Owner only. Blocks new listings/buys/offers/auctions. |
| `unpause` | `unpause()` | Owner only. Re-enables all operations. |

### Pause Behavior
- **Blocked when paused**: `list`, `buy`, `makeOffer`, `acceptOffer`, `makeCollectionOffer`, `acceptCollectionOffer`, `createAuction`, `bid`, `createDutchAuction`, `buyDutchAuction`, `createBundleListing`, `buyBundle`
- **Always allowed**: `cancelListing`, `cancelOffer`, `cancelCollectionOffer`, `cancelAuction`, `cancelDutchAuction`, `cancelBundleListing`, `settleAuction`, all view functions

**Events**: `PlatformFeeUpdated`, `FeeRecipientUpdated`, `Paused`, `Unpaused`

---

## 8. Payment Distribution

For every sale: `totalAmount → platformFee → royalty → seller`

1. **Platform fee**: `totalAmount * platformFeeBps / 10000`
2. **ERC-2981 royalty**: If NFT supports `royaltyInfo()`, pays `receiver` up to `remaining`
3. **Seller**: Gets the remainder

Supports both **native token** (`paymentToken = address(0)`) and **ERC-20** payments.

---

## 9. View Functions

| Function | Returns |
|----------|---------|
| `getListing(listingId)` | `Listing` struct |
| `getOffer(offerId)` | `Offer` struct |
| `getCollectionOffer(offerId)` | `CollectionOffer` struct |
| `getAuction(auctionId)` | `Auction` struct (includes `bidCount`) |
| `getDutchAuction(auctionId)` | `DutchAuction` struct |
| `getDutchAuctionCurrentPrice(auctionId)` | Current price (linear decay) |
| `getBundleListing(bundleId)` | `BundleListing` struct |
| `platformFeeBps()` | Current fee in bps |
| `feeRecipient()` | Fee recipient address |

---

## ERC-8004 Integration Notes

- ERC-8004 IdentityRegistry NFTs work with all marketplace features
- On transfer: `agentWallet` is automatically cleared by the registry's `_update()` override
- `MetadataSet` event emitted before `Transfer` event on every transfer
- Backend indexer listens for `Transfer` events to update DB ownership
- Tested: 32 integration tests covering all features with MockIdentityRegistry
