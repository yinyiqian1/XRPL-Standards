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
#### 2.1.2. Prohibiting transfering LPTokens that are frozen
### 2.2. AMM and Clawback
#### 2.2.1. Allow creation of AMM pool when tokens have enabled clawback
#### 2.2.2. New transaction to claw back from AMM pools  
This proposal introduces a new transaction type `AMMClawback` to allow asset issuers to claw back their assets from the AMM pool.  

Issuers can only claw back issued tokens in the AMM pool only if the `lsfAllowTrustLineClawback` flag is enabled. Attempting to do so without this flag set will result in an error code `tecNO_PERMISSION`.  

By designating the AMM account and holder account, this transaction will:  
- Claw back all LPTokens held by the specified holder account that are associated with the issuer from the specified AMM account.
- Initiate a two-asset withdrawal from the AMM account, resulting in:
  - The issuer's asset being returned to the issuer's account.
  - The non-issuer asset being transferred back to the holder's account

##### 2.2.2.1. Fields for AMMClawback transaction  

| Field Name          | Required?        |  JSON Type    | Internal Type     |
|---------------------|:----------------:|:-------------:|:-----------------:|
| `TransactionType`   |:heavy_check_mark:|`string`       |   `UINT16`        |  

`TransactionType` specifies the new transaction type `AMMClawback`. The integer value is 31. The recommended name is `ttAMMClawback`.

---

| Field Name          | Required?        |  JSON Type    | Internal Type     |
|---------------------|:----------------:|:-------------:|:-----------------:|
| `Account`           |:heavy_check_mark:|`string`       |   `ACCOUNT ID`    |  

`Account` designates the issuer of the asset being clawed back, and must match the account submitting the transaction.

---  
| Field Name          | Required?        |  JSON Type    | Internal Type     |
|---------------------|:----------------:|:-------------:|:-----------------:|
| `Holder`            |:heavy_check_mark:|`string`       |   `ACCOUNT ID`    |  

`Holder` specifies the holder account of the LP Token to be clawed back.

---

| Field Name          | Required?        |  JSON Type          | Internal Type     |
|---------------------|:----------------:|:-------------------:|:-----------------:|
| `AMMAccount`        |:heavy_check_mark:|`string`             |   `ACCOUNT ID`    |  

`AMMAccount` specifies the AMM account from which the transaction will withdraw assets after clawing back `Holder`'s LPtokens.

---  


##### 2.2.2.2. AMMClawback transaction example

```
{
  "TransactionType": "AMMClawback",
  "Account": "rPdYxU9dNkbzC5Y2h4jLbVJ3rMRrk7WVRL",
  "Holder": "rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B",
  "AMMAccount": "rp2MaZMQDpgAHwWbADaQMrmf4AD5JsPQUR",
  "Flags": 1,
  "Fee": 10
}
```

- Upon execution, this transaction enables the issuer `rPdYxU9dNkbzC5Y2h4jLbVJ3rMRrk7WVRL` to claw back up all LPTokens from holder `rvYAfWj5gh67oV6fW32ZzP3Aw4Eubs59B` associated with AMM account `rp2MaZMQDpgAHwWbADaQMrmf4AD5JsPQUR`.
- The transaction will result in the withdrawal of two corresponding assets from the AMM account:  
  - The asset issued by the `Account` will be returned to the issuer.
  - The other asset will be transferred back to the holder's wallet.
