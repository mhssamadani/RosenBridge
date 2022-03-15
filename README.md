# Rosen Bridge 
Rosen bridge is an Ergo-centric bridge enabling users to send and receive coins and tokens between Ergo and any other chain. The other chain, say chainX, however, should support multi-signature transactions or threshold signatures.

In this bridge, there is no need to deploy and utilize smart contracts on the other chains since the consensus on any action can be done on the Ergo platform by a set of Guards, resulting in a signed transaction (txn for Ergo or chainX). These transactions can be broadcasted to the other chain by any party including the Guards.

## Concept and Assumptions
-  This bridge utilizes a two-layer architecture. In the first layer, watchers are monitoring and report the events on the networks. Upon reaching a consensus on a particular event, the second layer is being notified. In the second layer, Guards will verify the event and create/sign the required transaction for ergo or chainX. 

- For any chainX, Guards hold a key for a multi-sig wallet (or a  share of a key for threshold signature). More sophisticated approaches are possible but not necessary. On the other hand, watchers do not have any keys.

- Guards utilize an m-out-of-n threshold/multisig scheme. So, the bridge will not be corrupted unless at least 'm' Guards are corrupted.

- Watchers monitor the chains, verify and report the events, and try to reach a consensus on the verified events. Then they notify the Guards of an approved event. Guards will verify the approved events and try to sign the required transaction.

- Watchers actively monitor the chains and trigger the events, while Guards only verify the approved events. This method allows this bridge to scale without increasing the Guards' computation. In theory, anyone can be a watcher and a set of predefined entities will work as Guards. The Watcher and Guard members can change over time.

- Note that the bridge is not relying on the security of the Watchers. In case of malfunctioning or corrupted Watchers, who create fraudulent events, the bridge is robust and sound since the Guards are individually validating the events.

- There should be a message/info/metadata support for the transaction of the other chain.

- To detect and avoid hard forks, double spending, known or unknown frauds, and race conditions this bridge is not sacrificing security for speed. For example, the watchers wait for a few confirmations before triggering the event, it takes a few Ergo blocks for the watchers to reach consensus, and the Guards are looking for more confirmations for validating the event.

- At the first glance, having a multi-sig wallet for other chains may look insufficient or not complex enough to satisfy an average user. In the end, all of these complicated methods rely on the same assumption as not having more than a threshold number of corrupted parties. On the other hand, since this bridge is only utilizing the smart contracts of Ergo, the whole process is auditable on Ergo and eliminates the need for having several smart contracts on different chains with different assumptions. This makes Rosen bridge pretty auditable and more secure than currently known bridges.

- Adding new chains is as easy as creating a wallet for the Guards, creating the Watchers, and setting some configurations.

# Actions

## Action 1- Transfering tokens from chainX to Ergo (e.g. Locking BTC on BTC/Minting rBTC on Ergo)

1- A user sends a payment to the bridge's multisignature wallet along with the metadata {destination Ergo address}. This tx can be generated by the UI.

2- Watchers periodically look for events from the multi-signature wallet.

3- In case of a received payment with enough confirmations, each Watcher creates a new 'event box' on the Ergo.

4- Any watcher can create an 'approved event box' if there are enough 'event boxes' for a particular event.

5- Guards are looking for 'approved event boxes'. In case of seeing an 'approved event box', each Guard verifies the validity of the event on chainX and signs the appropriate transaction on Ergo if the even it valid.

6- In case of enough signatures, the transaction is being broadcasted on Ergo.

7- User receives his rToken on Ergo.

8- Guards ensure that the broadcasted transaction is being mined and received enough confirmations.



## Action 2- Transfering rTokens from Ergo to chainX (e.g. Burning rBTC on Ergo/Unlcoking BTC on BTC)

1- Ergo user sends the fund to the Bank (or spends his box with the Bank box to create a new Bank box). There is metadata {destination chainX address} along with the transaction.

2- Watchers periodically look for events of the Bank box.

3- In case of a new payment with enough confirmations, each Watcher creates a new 'event box' on the Ergo. 

4- Any watcher can create an 'approved event box' if there are enough 'event boxes' for a particular event.

5- Guards are looking for 'approved event boxes'. In case of seeing an 'approved event box', each Guard verifies the validity of the event on Ergo and signs the appropriate transaction for chainX if the even it valid.

6- In case of enough signatures, the transaction is being broadcasted on chainX.

7- User receives his token on the chainX.

8- Guards ensure that the broadcasted transaction is being mined and received enough confirmations.



## Action 3- Transfering Ergo tokens to chainX (e.g. Locking SigUSD on Ergo/Minting rSigUSD on BTC)

Pretty similar to Action 2.


## Action 4- Transfering rTokens from chainX to Ergo (e.g. Burning rSigUSD on BTC/Unlocking SigUSD on Ergo)

Pretty similar to Action 1.

## Action 5- Transfering from chainX to chainY (e.g. Locking ADA on Cardano/Minting rADA on Ethereum)

It is somehow Action 1 followed by Action 3. This process can be done automatically without additional user intervention.

## Bridge fees

- fees are deducted on the Ergo side.

## Dust collection

In some cases, the transaction on Ergo does not spend all the 'event boxes'. In these cases, a dust collector should periodically spend these boxes. There might be other dust boxes as well to be collected.

# New chain integration

- Any chain with multisignature or threshold signature support can be added to the bridge network.

- For each new network the following should be implemented/deployed:

    - Watchers: to fetch info about the received payment in the multi-signature wallet.
    
    - New wallet for the Guards.
    
    - Transaction verification mechanism for the Guards.

    - Appropriate Bank boxes on Ergo

# Benefits

- A bridge to rule them all: it can be used to bridge any chain with multisignature/threshold signature support with Ergo. The additional development for a new chain is straightforward.

- Cheap: Almost all consensus transactions are done on the Ergo side.

- Cold storage enabled: A reasonable amount of funds on the other chain can be stored in multi-sig cold storage (protected by a separate set of keys). In case of a shortage, funds can be sent to the bridge's hot wallet.

- Security: Since the consensus is being reached on the Ergo side, the bridge is not prone to other chain's vulnerabilities to a good degree.

- Ease of integration: New chains can be integrated with the bridge without any knowledge of Ergo development and with the minimum efforts, described above.

- Auditable: Almost all of the process is being stored on the Ergo blockchain and can be audited by anyone. Also, since only Ergo smart contracts are used, the bridge's code and contracts, and developing new chain integrations are easier to audit in comparison with having a new code and smart contract for any new chain.

- Scalable: The Guards are only validating the approved events and do not waste time and resources to actively monitor several networks. On the other hand, lots of Watchers can be deployed for current and new chains without increasing the computations cost of the watchers. This makes the bridge more scalable and more robust to DoS attacks.


# Tokenization

In this section, we briefly introduce the 'Rosen' token which is the main driver of this bridge. More details will be available soon.

- The Guard set is predefined and consists of a few well-known and different entities. However, anyone can be a Watcher by staking a predefined amount of 'Rosen' tokens.
- By staking 'Rosen', a Watcher acquires a proportional amount of 'RosenEvent' token. Each 'even box' requires one 'RosenEvent' token. This token will be sent back to the watcher after the asset transfer is complete or will be collected by the bridge if the event is generated fraudulently.
- A watcher can unstake its 'Rosen' tokens by sending back 'RosenEvent' tokens. Therefore, losing 'RosenEvent' tokens due to fraudulent event reports will result in slashing the staked 'Rosen' tokens. This will incentivize the good behavior and punish the bad behavior of Watchers. At the same time, it is working as a mechanism to protect against DoS attacks.
- When the Guards are creating the final transaction for the user, they share the collected fee and emit new 'Rosen' tokens. So, being participated in a successful event reporting will benefit the Watcher in two ways: collecting the bridge's fee and receiving new 'Rosen' tokens. 
- Practically, any bridge path with a higher value of asset transfers will attract more watchers till the share of the fees and 'Rosen' emission is shrunk to a level that lower used paths become more profitable. Using this mechanism ensures the appropriate distribution of Watchers between different bridge paths.
- Watchers are needed to act independently and do the monitoring task individually. In practice, some Watchers may simply reply to the events created by other honest Watchers. To avoid this, there are a few well-known approaches which the simplest one is using a commit-reveal mechanism. However, a honeypot approach can be used in which fake events are being sent by some watchers and replayers will be punished by losing 'RosenEvent' tokens when they are found in the honeypot.


## Notes:

- This is a WIP. Any part of this design may change during the implementation.

- Please feel free to share your comments and ideas to improve this design.
