<br/>

## Name: Denial of Service

### Unique Identifier: EOSIO-WCR-114

### Vulnerability Rating: Medium

### Relationship: [CWE-400: Uncontrolled Resource Consumption](https://cwe.mitre.org/data/definitions/400.html)

## Background

In order to do anything on an EOSIO chain, a user needs to stake a proportional amount of system tokens, either to make computations (CPU), move data (NET) or sell them to store information (RAM).

Within EOSIO, blocks are created 500 milliseconds apart. To help ensure that Block Producers have enough time to propagate blocks around the world, there is a per-block processing time limit of 200 milliseconds (default settings) within which a Producer has to validate the block before broadcasting it to the network. This leaves 300 milliseconds for propagation across the network.


Some EOSIO chains come with a _free CPU leeway_ feature that allows accounts to consume more than their allotted resources if the overall network usage is low. Before this limit is reached, all users can freely transact on the network as it is not in **“congestion mode.”** Once this limit is passed, users are **throttled** back to their **pro-rata share** of the **total CPU-per-staked-EOS allotment**

<br/>

Example
> If there are **10,000 EOS tokens** being staked for **CPU** on **EOS** at a given moment, and Alice's account (an EOS whale) has **930 EOS tokens staked**, then Alice is guaranteed **9.3%** of the **total CPU capacity of the network**. If the network is being heavily used or suffering from a DoS attack, surpassing the **threshold** at which **rate limiting is activated**, Alice would not be able to exceed the guaranteed transaction rate allotted to her by her stake.

<br/>

Every EOS account has 2 separate values: **used CPU** and **total allowed CPU**. During a period when EOS is not in **congestion mode**, the **total allowed CPU** can fluctuate wildly. The actual CPU leeway multiplier is a chain parameter and it can happen that users are allowed to use 1000 times more CPU than in congestion mode. The image below demonstrates this fluctuation when the chain allows **20.6 ms.** of available compute _(left)_ one moment, to **11.8 ms.** of available compute _(right)_, the very next. 

<br/>

![rate limiting](images/rate_limiting.png)

> **Figure 5:** The CPU Variance means that an EOSIO account may show to have used more than 100% of its allotment

### Summary

Carefully crafted token contracts such as _EIDOS - Everyone is Denied of Service_ with airdrop patterns that entice rational actors on the EOS blockchain to generate nonutilitarian network traffic that causes performance degradation for all other users.

Degraded performance of EOS transactions due to an inadequate share of CPU staking in the face of a massive increase in network activity related 
to a token airdrop.

## Detailed Description

## Attack 

### Replication 

1. An EOS user sends a minimum of 0.0001 EOS to a Denial of Service Token Drop contract

2. The Denial of Service (DoS) token drop contract will then send a transaction returning the same amount of EOS back to the original sender 

3. The DoS token drop contract also **airdrops** a certain amount of a complimentary custom token as an incentive for the transaction to the original sender’s address


![sample airdrop](images/sample_dos_txn.png)

> **Figure 6:** Sample Airdrop Interaction Pattern



4. Since altcoin exchanges list the custom airdrop token with a direct fiat-based stablecoin pair i.e. USDT or USDC, rational actors with at least 0.0001 EOS, are incentivized to collect and attempt to sell the airdropped tokens for a non-zero amount of USDT / USDC



![cleared txns](images/cleared_exchange_txns.png)

> **Figure 7:** Cleared EOS/USDT Trades (Source: MXC Exchange)

5. To take advantage of this **arbitrage**, rational parties direct leased CPU towards the _airdrop contract_ as there is no penalty for repeated invocation of the airdrop claim, causing an EOS network flood

![airdrop claim](images/airdrop_claims.png)

> **Figure 8:** Number of airdrop _claim_ actions per hour skyrocket on EOS

6. The increased network activity causes the EOS network to enter **congestion mode**, which limits the number of transactions (TXNs) that users can broadcast to their **pro-rata share of total staked CPU**

7. Normal users that were previously able to process their transactions with their _staked CPU_ status quo are now unable to have their transactions processed, due to the same CPU stake now being considered too low an amount by the EOS protocol

| Note: _EOSIO protocol is behaving as expected, but congestion mode prevents users from having transactions processed that exceed their CPU stake_ |
| --- |

1. Such DoS network attacks can be expected to remain in motion as per game theory, until:
    * It is no longer profitable to collect the airdrops 
    * The sizeable _leases_ taken out of the **REX contract** expire (30 days) && the leasers **don’t renew** their _lease_

## Vulnerability

An attacker exploits the native EOSIO design choices by designing an airdrop contract that doesn’t take into account a minimum EOS threshold to be sent. Instead, the airdrop contract is _a spam-generating machine_ as it is designed to generate maximum transactions that cost users time but not money. 

Once users are refunded the EOS sent to the airdrop token contract, they are incentivized to send the EOS immediately back to the token contract due to the _incremental arbritage_ motive to do more actions rather than spend more money. In essence, such a contract is designed to test the capacity of EOSIO itself.

### Sample Code 
A sample **claim** action that encourages users to essentially spam the EOS network in return for a small monetary incentive

```c++
[[eosio::on_notify("eosio.token::transfer")]] void token::claim(name from, name to, eosio::asset quantity, std::string memo)
{
   if (to != get_self() || from == get_self()) 
         return;

   action{
         permission_level{get_self(), "active"_n},
         "eosio.token"_n,
         "transfer"_n,
         std::make_tuple(get_self(), from, quantity, std::string("Refund EOS"))
   }.send();              

   int elapsed = current_time_point().sec_since_epoch();

   if (elapsed > 1572595200) {
      asset supply = eosio::token::get_supply(get_self(), symbol_code("EIDOS"));    
      int64_t expected = (elapsed - 1572595200) * 25;

      asset balance = eosio::token::get_balance(get_self(), get_self(), symbol_code("EIDOS"));

      if (expected > 1000000000)
         expected = 1000000000;

      asset expected_supply = asset(expected, symbol("EIDOS", 4));
      expected_supply *= 10000;

      if (supply < expected_supply){
         asset claim = expected_supply-supply;

         action{
            permission_level{get_self(), "active"_n}, 
            get_self(),
            "issue"_n,
            std::make_tuple(get_self(), claim, std::string("Issue EIDOS"))
         }.send();

         balance += claim;

         claim = claim / 5;

         action{
            permission_level{get_self(), "active"_n},
            get_self(),
            "transfer"_n,
            std::make_tuple(get_self(), "eidosoneteam"_n, claim, std::string("Send to EIDOS Team."))
         }.send();              

         balance -= claim;

      }

      if (balance > asset(0, symbol("EIDOS", 4))) {
         if (balance <= asset(10000, symbol("EIDOS", 4)))
            balance = asset(1, symbol("EIDOS", 4));
         else   
            balance /= 10000;

         action{
            permission_level{get_self(), "active"_n},
            get_self(),
            "transfer"_n,
            std::make_tuple(get_self(), from, balance, std::string("Airdrop EIDOS"))
         }.send();              
      }
   }  
}  
```
> **Figure 9:** The first "transfer"_n refunds EOS while the final  "transfer"_n airdrops EIDOS the sample token

## Remediation
### Risk Reduction

* Victims like large cryptocurrency exchanges can respond to the network issue by increasing the amount of staked CPU in their wallets, to unblock customer transactions

* Users could stake enough resources for their purposes or lend resources from REX.

* Dapps can pay for their users' resources making use of the [`ONLY_BILL_FIRST_AUTHORIZER`](https://github.com/EOSIO/eos/issues/6332) protocol feature.

### Risk Mitigation

* Block Producers could **remove** the **80% cap** on **REX** rentals to allow the price to rent to rise high enough so that more EOS users are incentivized to move their EOS stake to REX

## References

- [Why Does My Account Run Out of CPU on EOS?](https://www.eoscanada.com/en/why-does-my-account-run-out-of-cpu-on-eos)
- [A Mysterious Airdrop Called EIDOS Is Clogging EOS to Make a Point](https://www.coindesk.com/a-mysterious-airdrop-called-eidos-is-clogging-eos-to-make-a-point)
- [EOS enters congestion mode due to EIDOS airdrop](https://blog.coinbase.com/eos-enters-congestion-mode-due-to-eidos-airdrop-3d3f82081074)



