# Vulnerabilities Found:

## 1. H-2 Lack of validation of ownership when fulfilling mint requests allowing NFTs to be stolen by anyone

### Problem Summary

Under `WeatherNft::fulfillMintRequest`, I noticed that the minting of NFTs does not validate weather the current caller is indeed the intended owner. Thus, this would allow attackers to monitor `requestId` being emitted in the events and intercept legitimate users by calling `WeatherNft::fulfillMintRequest` with the `requestId`, causing the NFT to be minted to the attacker instead.

### Mitigation

There are multiple solutions for this, the most straightforward being adding a require check such as the following to ensure that the current `msg.sender` is the same as the `UserMintRequest.user`.

```javascript
require(msg.sender ==
  s_funcReqIdToUserMintReq[requestId].user, WeatherNft__Unauthorized());
```

Another solution involves minting the NFT to `UserMintRequest.user` instead of `msg.sender`.

```diff
-_mint(msg.sender, tokenId);
+_mint(s_funcReqIdToUserMintReq[requestId].user, tokenId);
```

## 2. H-3 Lack control of NFT minting allowing unlimited NFTs to be minted for the same requestId

### Problem Summary

Within the same `WeatherNft::fulfillMintRequest` function, there are also no controls to prevent multiple executions with the same `requestId`, allowing unlimited NFTs.

### Mitigation

We can add an additional mapping to track which `requestId`s have already been minted and fulfilled.

```diff
 function fulfillMintRequest(bytes32 requestId) external {
+    require(!fulfilled[requestId], "Already fulfilled");
+    fulfilled[requestId] = true;
     // ... existing logic ...
 }
```

# Vulnerabilities Missed

## H-1 Contract receives ETH payment but lacks withdraw functions causing any ETH sent to be locked permanently

### Review

This is something very obvious that I did not think about checking. In future audits I have to take note of this as it is extremely detrimental to the project yet very simple and obvious to check. Additionally this was caught by Slither and Aderyn.

## H-4 Inadequate handling of Chainlink Functions request errors can lead to user's native tokens being stuck.

### Problem Summary

The root of the problem lies in the following lines under `WeatherNft::fulfillMintRequest`:

```javascript
    bytes memory response = s_funcReqIdToMintFunctionReqResponse[requestId].response;
    bytes memory err = s_funcReqIdToMintFunctionReqResponse[requestId].err;

    require(response.length > 0 || err.length > 0, WeatherNft__Unauthorized());

    if (response.length == 0 || err.length > 0) {
        return;
    }
```

This function lacks a mechanism to allow the user to reclaim their initial native token payment when an explicit error is returned by Chainlink Functions, or when the request simply fails to fulfill at all. This results in the user's funds becoming permanently locked in the contract without receiving the NFT.

### Review

This is extremely important to take note of next time, especially when conducting audits on projects that use oracles. There needs to be safety mechanism in place in the event that oracles return erroneous data.

## H-5 Unrestricted `performUpkeep` Function ALlows Attackers to Drain Chainlink Automation Funds

### Problem Summary

The `performUpkeep` function allows any external address to call it with a valid tokenId, without any access controls or heartbeat checks. Thus an attacker can repeatedly call this function to drain all LINK from the subscription and cause a DOS on the automated weather updates.

### Review

This would have been harder to catch for me given that this is my first time auditing a project that makes use of oracles. In the future I need to take note of how the contracts interact with the oracle, and how the oracle functions and how it is being funded.
