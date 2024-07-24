<pre>
Title:       <b>AMMClawback</b>
Type:        <b>draft</b>

Author:      <a href="mailto:shawnxie@ripple.com">Shawn Xie</a>
             <a href="mailto:yqian@ripple.com">Yinyi Qian</a>
Affiliation: <a href="https://ripple.com">Ripple</a>
</pre>

#  AMMClawback

## Abstract

The AMMClawback amendment empowers token issuers on the XRP Ledger to regulate tokens in an effort to prevent misuse by blacklisted accounts. This document proposes enhancements to the interactions between frozen assets and AMM pools, as well as new clawback mechanism to enable issuer claw back from wallets who have deposited into AMM pools. This proposal will ensure that blacklisted token holders cannot transfer unauthorized funds and will significantly improve use cases such as stablecoins, where ensuring regulatory compliance is essential.

## 1. Overview
This proposal introduces new improvements on how AMM interacts with frozen asset and clawback.

### 1.1. AMM and Frozen Asset
Tokens in frozen trustlines (either global or individual) are currently able to interact with AMM pools, which leaves opportunities for blacklisted accounts to maliciously move funds elsewhere through the AMM. To prevent these behaviors, this proposal introduces the following:
* Prohibit a wallet from depositing new tokens (single-sided and double-sided) into an AMM pool if at least one of the tokens the wallet owns has been frozen (either individually or globally) by the token issuer 
* Prohibit wallets from transfering `LPTokens` to another address if one of the tokens in the pool has been frozen (either individually or globally) by the token issuer

### 1.2. AMM and Clawback
Currently, accounts that have enabled clawback cannot create AMM pools. This proposal introduces the following:
* Allow an account to create AMM pool even if one of the tokens has enabled `lsfAllowTrustLineClawback`. However, token issuers will not be allowed to claw back from the AMM account using the `Clawback` transaction.
* Introduce a new `AMMClawback` transaction to allow token issuers to exclusively claw back from AMM pools that has one of their tokens, according to the current proportion in the pool.


## 2. Specification
### 2.1. AMM and Frozen Asset 
#### 2.1.1. Prohibiting depositing new tokens

The AMMClawback amendment introduces changes to the behavior of the `AMMDeposit` transaction when tokens in trustlines interact with Automated Market Maker (AMM) pools. 

Assume we have created an Automated Market Maker (AMM) pool with two assets: A and B. Currently, asset A is frozen by individual freeze. The following table outlines whether specific scenarios are allowed or prohibited before and after the amendment:

| Scenario                | Before Amendment  | After Amendment       |
|-------------------------|-------------------|-----------------------|
| Double-Asset Deposit    | Prohibited        | Prohibited            |
| Only Deposit A (frozen) | Prohibited        | Prohibited            |
| Only Deposit B          | Allowed           | Prohibited            |  

As illustrated in the table above, the primary change is that when one asset in the AMM pool is frozen, depositing the other asset is no longer allowed. This means that deposits are prohibited for the non-frozen asset when its paired asset is frozen.

#### 2.1.2. Prohibiting transfering LPTokens that are frozen
### 2.2. AMM and Clawback
#### 2.2.1. Allow creation of AMM pool when tokens have enabled clawback
Currently, when clawback is enabled for the issuer account by setting `lsfAllowTrustLineClawback` flag, `AMMCreate` is prohibited against this issuer. After the AMMClawback amendment, `AMMCreate` is allowed for clawback-enabled issuer. But the issuer can not clawback from the AMM account using `Clawback` transaction. `AMMClawback` transaction is needed for the issuer to clawback from an AMM account.  

**Example Illustrating the AMMClawback Amendment:**  

Suppose an issuer has enabled clawback by setting the `lsfAllowTrustLineClawback` flag through an `AccountSet` transaction. Additionally, two trustlines have been established between the holder and the issuer for currencies A and B 

- **Pre-Amendment Behavior:**  
    - If the holder attempts to create an AMM pool (`AMMCreate`) using the pair of trustline assets A and B associated with the issuer, the transaction will fail with a `tecNO_PERMISSION` error.

- **Post-Amendment Behavior:**  
    - After the AMMClawback amendment, if the holder submits an `AMMCreate` transaction to create an AMM pool with assets A and B, the transaction will be successful.
    - However, if the issuer wants to clawback assets from the AMM account, they must use the `AMMClawback` transaction instead of the regular `Clawback` transaction.  

This change allows for the creation of AMM pools with clawback-enabled issuers while introducing a new transaction type (`AMMClawback`) for issuers to clawback assets from AMM accounts.


#### 2.2.2. New transaction to claw back from AMM pools

