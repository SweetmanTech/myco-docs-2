# Rewards

`/api/zora/rewards`

## Getting Started

To start using the Myco Rewards API, follow these steps:

1. Make a Request

   - Use the provided examples in the [How to Use](#how-to-use) section to make your first API call.

2. Handle the Response

   - Refer to the [Return Values](#definitions) section to understand the response structure.

## Purpose

The Myco Rewards API provides builders with insights into the Zora rewards protocol. It enables creators and developers to:

1. Track rewards earned from mints on Zora
2. Monitor key metrics such as Daily Active Mints and top collectors
3. Access total rewards earned across the Zora ecosystem

By leveraging this API, developers can build features that give creators visibility into their earnings and engagement on the Zora platform. This aligns with Zora's mission to provide the best tools for creators to monetize their work onchain, as outlined in their [Protocol Rewards documentation](https://docs.zora.co/contracts/rewards).

Some potential use cases include:

- Creating dashboards for creators to visualize their earnings
- Implementing leaderboards for top collectors
- Building analytics tools to track minting activity and reward distribution

The Myco Rewards API serves as a bridge between Zora's protocol rewards system and developer applications, facilitating the creation of value-added services within the Zora ecosystem.

## Demo Video

Watch this demo video to see the Myco Rewards API in action:

<iframe width="560" height="315" src="https://www.youtube.com/embed/JSUeTBq3GzE" frameborder="0" allowfullscreen></iframe>
This video demonstrates how to use the Myco Rewards API to access and visualize data from the Zora rewards protocol, showcasing its potential for creating powerful analytics tools and dashboards for creators.

## How to Use

`https://myco.wtf/api/zora/rewards`

The Myco Rewards API allows you to retrieve reward information for a specific Ethereum address. Below are examples of how to perform a GET request using different programming languages and tools.

### JavaScript (Fetch API)

```javascript
const address = "0x33912a0d6bEFf5Fb8e5B70688CE858D5e7E8104E";
const url = `https://myco.wtf/api/zora/rewards?address=${address}`;

fetch(url)
  .then((response) => response.json())
  .then((data) => {
    console.log("Rewards data:", data);
    // Process the rewards data here
  })
  .catch((error) => {
    console.error("Error fetching rewards data:", error);
  });
```

### Python (Requests Library)

```python
import requests

address = '0x33912a0d6bEFf5Fb8e5B70688CE858D5e7E8104E'
url = f'https://myco.wtf/api/zora/rewards?address={address}'

response = requests.get(url)

if response.status_code == 200:
    data = response.json()
    print('Rewards data:', data)
    # Process the rewards data here
else:
    print('Error fetching rewards data:', response.status_code, response.text)
```

### cURL

```bash
curl -X GET "https://myco.wtf/api/zora/rewards?address=0x33912a0d6bEFf5Fb8e5B70688CE858D5e7E8104E"
```

## Parameters

| Parameter | Type                                                   | Required | Default | Description                         |
| --------- | ------------------------------------------------------ | -------- | ------- | ----------------------------------- |
| `address` | [Address](https://viem.sh/docs/glossary/types#address) | No       | `null`  | The Ethereum address of the creator |

### Return Values

The API returns a JSON object with the following structure:

| Field      | Type   | Description                                      |
| ---------- | ------ | ------------------------------------------------ |
| `message`  | string | Status message of the API call                   |
| `address`  | string | The Ethereum address of the creator              |
| `response` | object | An object containing detailed reward information |

#### Response Object

| Field                       | Type   | Description                                               |
| --------------------------- | ------ | --------------------------------------------------------- |
| `zoraCreateReferralRewards` | number | Rewards earned from Zora create referrals (**sparks** ✧)  |
| `zoraMintReferralRewards`   | number | Rewards earned from Zora mint referrals (**sparks** ✧)    |
| `zoraFirstMinterRewards`    | number | Rewards earned as first minter on Zora (**sparks** ✧)     |
| `zoraCreatorRewards`        | number | Rewards earned as a creator on Zora (**sparks** ✧)        |
| `baseCreateReferralRewards` | number | Rewards earned from Base create referrals (**sparks** ✧)  |
| `baseMintReferralRewards`   | number | Rewards earned from Base mint referrals (**sparks** ✧)    |
| `baseFirstMinterRewards`    | number | Rewards earned as first minter on Base (**sparks** ✧)     |
| `baseCreatorRewards`        | number | Rewards earned as a creator on Base (**sparks** ✧)        |
| `totalRewards`              | number | Total rewards earned across all categories (**sparks** ✧) |
| `events`                    | array  | An array of reward event objects                          |

#### Event Object

Each object in the `events` array contains:

| Field       | Type   | Description                                |
| ----------- | ------ | ------------------------------------------ |
| `event`     | string | Type of reward event                       |
| `address`   | string | Ethereum address associated with the event |
| `timestamp` | string | ISO 8601 formatted timestamp of the event  |
| `points`    | number | Number of points earned in the event       |
| `metadata`  | object | Additional metadata about the event        |

#### Metadata Object

The `metadata` object contains various fields depending on the event type. Common fields include:

| Field                     | Type   | Description                                          |
| ------------------------- | ------ | ---------------------------------------------------- |
| `from`                    | string | Address of the sender                                |
| `zora`                    | string | Zora contract address                                |
| `creator`                 | string | Address of the creator                               |
| `feeType`                 | string | Type of fee (e.g., "firstMinter")                    |
| `network`                 | string | Network where the event occurred                     |
| `tokenId`                 | string | ID of the token involved                             |
| `quantity`                | string | Quantity of tokens                                   |
| `uniqueId`                | string | Unique identifier for the event                      |
| `collector`               | string | Address of the collector                             |
| `zoraReward`              | string | Amount of Zora reward (**sparks** ✧)                 |
| `blockNumber`             | string | Block number of the transaction                      |
| `transactionHash`         | string | Hash of the transaction                              |
| `firstMinter`             | string | Address of the first minter                          |
| `mintReferral`            | string | Address of the mint referral                         |
| `creatorReward`           | string | Reward amount for the creator (**sparks** ✧)         |
| `createReferral`          | string | Address of the create referral                       |
| `collectionAddress`       | string | Address of the collection involved                   |
| `firstMinterReward`       | string | Reward amount for the first minter (**sparks** ✧)    |
| `mintReferralReward`      | string | Reward amount for the mint referral (**sparks** ✧)   |
| `createReferralReward`    | string | Reward amount for the create referral (**sparks** ✧) |
| `protocolRewardsContract` | string | Address of the protocol rewards contract             |

### Example Response

```json
{
  "message": "success",
  "address": "0xcfBf34d385EA2d5Eb947063b67eA226dcDA3DC38",
  "response": {
    "zoraCreateReferralRewards": 0,
    "zoraMintReferralRewards": 0,
    "zoraFirstMinterRewards": 0,
    "zoraCreatorRewards": 0,
    "baseCreateReferralRewards": 0,
    "baseMintReferralRewards": 0,
    "baseFirstMinterRewards": 0,
    "baseCreatorRewards": 0,
    "totalRewards": 444666,
    "events": [
      {
        "event": "RewardsDeposit-firstMinter-baseSepolia",
        "address": "0xcfBf34d385EA2d5Eb947063b67eA226dcDA3DC38",
        "timestamp": "2024-10-12T10:54:51.728Z",
        "points": 111,
        "metadata": {
          "from": "0x78aeF8d0bdbA272F34d86bF6B8ba96aa54584932",
          "zora": "0x12125c8a52B8E4ed1A28e1f964023b4477f11300",
          "creator": "0xcfBf34d385EA2d5Eb947063b67eA226dcDA3DC38",
          "feeType": "firstMinter",
          "network": "baseSepolia",
          "tokenId": "1",
          "quantity": "1",
          "uniqueId": "baseSepolia-0x01cfddb9cfb3cc0b8da6b2595c557cebb3707936ef095e9fd459579fcc9315a9-0x8-firstMinter",
          "collector": "0x15D46DAfb26B788Ee37e981c6569318673404Bbc",
          "zoraReward": "111000000000000",
          "blockNumber": "0xabc848",
          "firstMinter": "0xcfBf34d385EA2d5Eb947063b67eA226dcDA3DC38",
          "mintReferral": "0x15D46DAfb26B788Ee37e981c6569318673404Bbc",
          "creatorReward": "333000000000000",
          "createReferral": "0x12125c8a52B8E4ed1A28e1f964023b4477f11300",
          "transactionHash": "0x01cfddb9cfb3cc0b8da6b2595c557cebb3707936ef095e9fd459579fcc9315a9",
          "firstMinterReward": "111000000000000",
          "mintReferralReward": "111000000000000",
          "createReferralReward": "111000000000000",
          "protocolRewardsContract": "0x7777777f279eba3d3ad8f4e708545291a6fdba8b"
        }
      }
      // ... more event objects
    ]
  }
}
```

## Related Documentation

- [Score](./score.md)
- [Profile](./profile.md)
- [Zora Protocol Rewards](https://docs.zora.co/contracts/rewards)

## Frequently Asked Questions (FAQ)

### 1. **Can I retrieve rewards for multiple addresses in a single request?**

Currently, the `/api/zora/rewards` endpoint only supports fetching rewards for one address at a time. If you need to retrieve data for multiple addresses, you will need to make separate requests for each address.

### 2. **How often is the rewards data updated?**

The rewards data is updated in real-time as transactions occur on the Zora protocol. However, there might be slight delays in data propagation.

### 3. **What units are the rewards amounts in?**

Rewards amounts are denominated in wei, the smallest unit of ether. Ensure you convert them to standard units (e.g., ETH) as needed.

### 4. **Is there a sandbox environment available for testing?**

Yes, you can use the Base Sepolia and Zora Sepolia networks for testing purposes.

### 5. **Who should I contact for support?**

For any issues or questions, please reach out to our [support team](mailto:support@myco.wtf) or visit our [GitHub repository](https://github.com/SweetmanTech/myco-api) for more information.
