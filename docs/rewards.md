# Rewards

`/api/zora/rewards`

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

` https://myco.wtf/api/zora/rewards`

To use the Myco Rewards API, you can make a GET request to the endpoint with the creator's address as a parameter. Here's an example of how to do this using JavaScript and the Fetch API:

### Example GET Request

javascript

```
const address = '0x33912a0d6bEFf5Fb8e5B70688CE858D5e7E8104E'; // newtroarts.eth
const url = https://myco.wtf/api/zora/rewards?address=${address};
fetch(url)
.then(response => response.json())
.then(data => {
    console.log('Rewards data:', data);
    // Process the rewards data here
})
.catch(error => {
    console.error('Error fetching rewards data:', error);
});
```

## Definitions

[List of parameters, default values, and return values will be added here]
