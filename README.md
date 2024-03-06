# L2 Auction SDK

SDK for L2 auction bidders.

## Table of Contents

- [RPC Endpoints](#rpc-endpoints)
- [Bundle JSON Requests and Responses](#json-requests-and-responses)
- [L1 Bridge](#l1-bridge)
- [WETH](#weth)
- [Auction Contracts](#auction-contracts)
- [Bidder Contracts](#bidder-contracts)
- [Bundler Examples](#bundler-examples)
- [Contributing](#contributing)
- [License](#license)


## RPC Endpoints

- **L2 RPC (TESTNET):**
  - Description: L2 Node RPC (Testnet)
  - URL: `[https://holesky-api.securerpc.com/l2](https://holesky-api.securerpc.com/l2/)`
  - Methods: eth_* 
  - ChainId: 42169

- **Beta bundle RPC (Testnet):**
  - Description: Beta bundle submission RPC
  - URL: `[https://holesky-api.securerpc.com/v2](https://holesky-api.securerpc.com/v2)`
  - Method: mev_sendBetaBundle
  - Parameters:
    - `txs`: List of txs as bundle e.g. [0x2323...,]
    - `slot`: slot number e.g. "11282389"
  - ChainId: 17000

## Bundle JSON Requests and Responses

```json
// Example JSON request
{
    "jsonrpc": "2.0",
    "method": "mev_sendBetaBundle",
    "params": [
      {
        "txs": [0x... ],
        "slot": "1001"
      }
    ],
    "id": 8
}

// Example JSON response
{
    'jsonrpc': '2.0',
    'id': 1,
    'method': 'mev_sendBetaBundle',
    'result': '0x79e5cba7876f532218ac35a357209800be2362dd2e3f1e6dc5974698f0d7cee4'
}
```

## L1 Bridge

Fund L2 address by sending eth to the bridge address.

### Deployed Address (Testnet)
```bash
L1_BRIDGE="0x3Ae5Ca0B05bE12d4FF9983Ed70D86de9C34e820C"
```

## WETH

### Deployed Address (Testnet)
```bash
WETH="0x4200000000000000000000000000000000000006"
```

## Auction Contracts

### Deployed Addresses (Testnet)
```bash
AUCTIONEER="0xD88e271AD39dAb4DEB0f7f9475d993ed4Bf5029b"
SETTLEMENT="0x7Ac1A452B59114Fb1E67470720343A2e9AE18297"
```

### Registering a bidder

Only registered bidders can participate in the auction. Operators can onboard new bidders through the contract. 

To check for bidderId when registered, call `IdMap` on the contract:
```solidity
function IdMap(address bidder) external view returns (uint8 id);
```

### Packing a bid

Bids are packed by price, amount, bidderId
```solidity
    /**
     * @dev Packed Bid details into a uint256 for submission.
     *
     * @param bidPrice Price per item.
     * @param itemsToBuy Items to buy in the auction.
     * @param bidderId Id for bidder
     * @return packedBid for auction submission
     */
    function packBid(uint256 bidPrice, uint256 itemsToBuy, uint256 bidderId)
        external
        pure
        returns (uint256 packedBid);
```

### Winning bid info

After an auction is closed, bidders can query their bid results:
```solidity
    /**
     * @dev Retrieve information about a bidder after auction settlement.
     *
     * @param slot The slot identifier of the auction.
     * @param bidder The address of the bidder for whom information is requested.
     * @return itemsBought The number of items bought by the bidder in the specified auction.
     * @return amountOwed The amount owed by the bidder for the items bought in the specified auction.
     *
     * Requirements:
     * - The auction must have been settled.
     * - The provided `bidder` address must be valid and have participated in the auction.
     *
     */
    function getBidderInfo(uint256 slot, address bidder)
        external
        view
        returns (uint120 itemsBought, uint128 amountOwed);
```

## Bidder Contracts

A minimal viable bidder is provided below:
```solidity
pragma solidity 0.8.20;

import {Auctioneer} from "github.com/manifoldfinance/auctioneer/Auctioneer.sol";
import {SettlementHouse} from "github.com/manifoldfinance/auctioneer/SettlementHouse.sol";
import {WETH} from "solmate/tokens/WETH.sol";

contract MockBidder {
    uint256[] public bids;
    Auctioneer auctioneer;
    SettlementHouse house;
    WETH weth;

    constructor(WETH _weth, address _auctioneer, address settlement) {
        weth = _weth;
        auctioneer = Auctioneer(_auctioneer);
        house = SettlementHouse(settlement);
        weth.approve(_auctioneer, type(uint256).max);
    }

    function setBids(uint256[] memory newBids) public {
        bids = newBids;
    }

    function getBid(uint256) external view returns (uint256[] memory packedBids) {
        return bids;
    }

    function submit(uint256 slot, uint256 amount, bytes32[] calldata hashes) external {
        auctioneer.approve(address(house), slot, amount);
        house.submitBundle(slot, amount, hashes);
    }
}
```

## Bundler Examples

2 example bundlers are given:
- [Python bundler](https://github.com/MEV-Protocol/beta-bundles-py) - employs a deployed bidder contract for continuous automated bidding, while listening for auction close event, then submits the bundle
- [Go bundler](https://github.com/manifoldfinance/auction-bidder) - uses a manual bidder before submitting bundles

## Contributing

Contributions are welcome! If you find any issues or have suggestions for improvements, feel free to open an issue or submit a pull request.

## License

This project is licensed under the [MIT License](LICENSE).