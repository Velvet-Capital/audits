# **Velvet Capital Audit Competition on Hats.finance** 


## Introduction to Hats.finance


Hats.finance builds autonomous security infrastructure for integration with major DeFi protocols to secure users' assets. 
It aims to be the decentralized choice for Web3 security, offering proactive security mechanisms like decentralized audit competitions and bug bounties. 
The protocol facilitates audit competitions to quickly secure smart contracts by having auditors compete, thereby reducing auditing costs and accelerating submissions. 
This aligns with their mission of fostering a robust, secure, and scalable Web3 ecosystem through decentralized security solutions​.

## About Hats Audit Competition


Hats Audit Competitions offer a unique and decentralized approach to enhancing the security of web3 projects. Leveraging the large collective expertise of hundreds of skilled auditors, these competitions foster a proactive bug hunting environment to fortify projects before their launch. Unlike traditional security assessments, Hats Audit Competitions operate on a time-based and results-driven model, ensuring that only successful auditors are rewarded for their contributions. This pay-for-results ethos not only allocates budgets more efficiently by paying exclusively for identified vulnerabilities but also retains funds if no issues are discovered. With a streamlined evaluation process, Hats prioritizes quality over quantity by rewarding the first submitter of a vulnerability, thus eliminating duplicate efforts and attracting top talent in web3 auditing. The process embodies Hats Finance's commitment to reducing fees, maintaining project control, and promoting high-quality security assessments, setting a new standard for decentralized security in the web3 space​​.

## Velvet Capital Overview

Intent Operating System streamlining portfolio management and make DeFi social

## Competition Details


- Type: A public audit competition hosted by Velvet Capital
- Duration: 2 weeks
- Maximum Reward: $69,870.71
- Submissions: 113
- Total Payout: $40,427.19 distributed among 16 participants.

## Scope of Audit

[Velvet Capital Repo](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77)

## High severity issues


- **DepositBatch Function Allows Draining of Approvers' Token Balances with Malicious Calldata**

  There's a vulnerability in the DepositBatch, WithdrawBatch, and EnsoHandler contracts, which allows for the draining of tokens due to improper input validation. Specifically, the DepositBatch.multiTokenSwapAndTransfer() function can be exploited through delegate calls via Enso's SafeEnsoShortcuts integration. This exploit enables an attacker to call any command, including the transferFrom function of the USDT contract, using malicious calldata.

In a typical attack scenario, a user (User A) approves the DepositBatch contract to spend their USDT tokens. The attacker then invokes the vulnerable function with maliciously crafted calldata, causing User A's USDT balance to be transferred to the attacker's wallet. The exploitation is facilitated by the fact that the contracts making the delegate calls are considered as the callers, thereby bypassing approval checks.

A proof of concept (PoC) demonstrating the exploit was provided. The recommended mitigation involves implementing a whitelisting mechanism for target contracts and function selectors. Further collaboration between the reporting user and the maintainers led to a refined PoC and eventual resolution of the issue. The changes have been committed and can be reviewed in the provided commit link.


  **Link**: [Issue #78](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/78)

## Medium severity issues


- **Unprotected ERC20 permit in VaultManager.sol allows front-running DoS attacks on deposits**

  The `VaultManager.sol` contract has an unprotected ERC20 permit functionality, which can be exploited for Denial-of-Service (DoS) attacks through front-running. When users grant permits for token transfers, anyone can execute the permit if they catch the transaction details in the mempool. This can lead to the front-runner consuming the permit before the intended transaction completes, causing the original transaction to fail due to invalidated signatures. The risk is exacerbated when multiple permits are batched, as a front-runner can target the last permit in the batch, forcing unnecessary gas expenditure on failed transactions. A recommended solution is wrapping permit calls in a try/catch block to prevent the entire call chain from reverting in case of a reused signature.


  **Link**: [Issue #5](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/5)


- **HighWaterMark reset to 0 on first deposit does not function as intended**

  During the first deposit, the watermark should be set to 0, but this can never happen if the watermark is already greater than 0. This is due to the current `_updateHighWaterMark` function, which only updates the watermark if the new price is higher than the existing high water mark. To align with the protocol's intent, the function should be modified to set the highWatermark to the current price regardless of its value. Specifically, updating the function logic to always change the highWatermark to the new price will fix this issue. The required adjustments have been made and the changes can be reviewed at the provided link.


  **Link**: [Issue #45](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/45)


- **Admin Fee Parameter Changes Can Cause Incorrect Minting of Accrued Fees**

  Each portfolio is connected with a fee module that charges and mints protocol and management fees based on token supply and configurations, which can be altered by the ProtocolOwner. This leads to a situation where changing crucial fee parameters without accounting for previously accrued fees can result in the protocol minting incorrect fee amounts. The `VaultManager` triggers the `_chargeFees()` function during deposits and withdrawals, which calculates fees based on total supply and time intervals. However, if no transactions occur for a while, accrued fees might not be minted correctly. Adjusting fee parameters without minting these accrued fees could lead to inaccuracies in fee collection. The suggested solution involves calling `feeModule._chargeProtocolAndManagementFees()` before updating fee parameters.


  **Link**: [Issue #47](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/47)


- **Failed Rebalancing Due to Dust Tokens Can Cause Permanent Denial of Service**

  The `updateWeights` function in the Rebalancing contract, used by the asset manager to update asset weights, is found to have a critical flaw. The process involves pulling funds from the vault to the swap handler, performing a swap using the EnsoHandler contract, checking for leftover tokens in the handler, and verifying vault token balances. The strict verification process in step 3 often fails because some leftover (dust) tokens remain post-swap. Additionally, if the swap handler is pre-funded with a small token amount, it leads to an accumulation of excess tokens, causing the operation to revert. This creates a potential Denial of Service (DoS) scenario, making the protocol vulnerable to being permanently blocked from updating weights, which could render the protocol insolvent. The proposed fix suggests transferring any leftover tokens from the EnsoHandler back to the vault instead.


  **Link**: [Issue #61](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/61)

## Low severity issues


- **USDT Approval in multiTokenSwapAndTransfer Fails on Ethereum Mainnet**

  The `multiTokenSwapAndTransfer` function in `DepositBatch` has an issue with the plain `Approve` ERC20 function. Specifically, approvals for the USDT token on the Ethereum Mainnet will fail due to its requirement for allowances to be set to zero before modifying them. It's recommended to set the allowance to zero before setting it to a non-zero value or to use a safe increase allowance method. This issue is currently not affecting deployments on BSC and Arbitrum but may affect future multi-chain deployments.


  **Link**: [Issue #3](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/3)


- **VaultManager Fails to Enforce Minimum Mint Amount Properly**

  The VaultManager fails to properly enforce the `_minMintAmount` when users deposit funds. The minimum mint amount is checked before fees are applied, potentially allowing users to receive fewer tokens than specified. This breaches a core invariant of the system. A code adjustment has already been made to address this issue.


  **Link**: [Issue #31](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/31)


- **TokenWhitelistManagement lacks remove operation, needs validation or implementation**

  The `TokenWhitelistManagement` module only supports adding tokens during initialization and lacks a function to remove tokens despite having an event for token removal. This discrepancy suggests either implementing token removal or ensuring a validation check when `_tokenWhitelistingEnabled` is true and `_whitelistTokens` is empty. The issue has been resolved in the latest commit.


  **Link**: [Issue #33](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/33)


- **Claim Function Reverts Entirely if Single Token Transfer Fails**

  The `claimRemovedTokens` function allows users to claim their share of tokens removed in previous snapshots. However, if any single transfer operation within the function fails, the entire process reverts, preventing users from claiming any tokens and potentially leading to financial losses and poor user experience.


  **Link**: [Issue #60](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/60)


- **Issues with Rebasing Tokens in TokenExclusionManager Affecting Claims and Balances**

  The balance discrepancies of rebasing tokens when they are removed from a portfolio can cause complications. With increasing rebase tokens, an excess accumulates, while decreasing rebase tokens may leave insufficient funds, leading to failed user claims. Solutions include using current balances for claims and adding a function to transfer tokens from the exclusion manager.


  **Link**: [Issue #74](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/74)


- **Edge Case in Claiming Removed Tokens May Cause Out of Gas Errors**

  A potential large gap between `lastClaimedUserId` and `_currentSnapshotId` can cause an out-of-gas error when claiming removed tokens. This issue arises when the code executes repeated claim attempts due to the `hasInteractedWithId` flag being gamed by a small transfer, triggering extensive looping. Implementing transaction iteration limits or batching claims could mitigate this.


  **Link**: [Issue #75](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/75)


- **Protocol Does Not Restrict Token Transfers or Removals During Pause State**

  The protocol restricts deposits and withdrawals during its pause state but doesn't prevent token transfers via `transfer()` or token removal functions in the asset management contract. This oversight undermines the purpose of pausing mechanisms intended to mitigate security issues. Recommendations include adding pause checks before token transfers and token removal functions.


  **Link**: [Issue #102](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/102)

## Minor severity issues


- **Dynamic Array Indexed in UpdatedTokens Event Returns Inaccurate Hashed Data**

  When the `indexed` keyword is used for reference type variables like dynamic arrays or strings, it returns their hash instead of the actual data. This affects the Rebalancing contract's `UpdatedTokens` event, causing front-end applications to receive meaningless hashes rather than the expected parameters. Removing `indexed` resolves the issue.


  **Link**: [Issue #25](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/25)


- **Potential Token Balance Transfer Issue with Edge-Case Portfolios and Malicious Managers**

  Tokens with a special condition to transfer the full balance when transferring `amount == type(uint256).max` can disrupt accounting or be exploited by malicious managers. This can cause an inflated minting of portfolio shares or enable zero-balance deposits, potentially breaking the portfolio. A recommended fix is to disallow using `type(uint256).max` as a deposit amount.


  **Link**: [Issue #66](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/66)


- **Wrong Event Emission in `upgradeTokenExclusionManager()` of `PortfolioFactory` Contract**

  The `upgradeTokenExclusionManager()` function in the `PortfolioFactory` contract is emitting the wrong event, `UpgradePortfolio`, instead of an event specific to upgrading the token exclusion manager. This misleads event-listening programs into thinking a portfolio is being upgraded. The function's natspec description and corresponding event are also incorrectly aligned, mirroring the `upgradePortfolio()` function. A suggested fix includes defining and using a correct event for the token exclusion manager upgrade.


  **Link**: [Issue #77](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/77)


- **Ownership Transfers Not Updating Owner Information in Portfolio List**

  When a portfolio transfers ownership to another entity, the `getPortfolioList` function returns outdated owner information. The `PortfolioInfolList` is not updated to reflect the new owner. It's recommended to override the `transferOwnership` function to update the owner in `PortfolioInfolList`. This issue has been addressed and resolved.


  **Link**: [Issue #83](https://github.com/hats-finance/Velvet-Capital-0x0bb0c08fd9eeaf190064f4c66f11d18182961f77/issues/83)



## Conclusion

In summary, the Hats.finance audit competition for Velvet Capital conducted over two weeks with a reward pool of nearly $70,000 succeeded in identifying 113 security vulnerabilities, with a total payout of $40,427.19 distributed among 16 participants. High-severity issues included a critical vulnerability in the DepositBatch function allowing malicious actors to drain token balances through improper calldata validation. Medium-severity issues ranged from unprotected ERC20 permit functionality enabling front-running attacks to improper handling of admin fee parameter changes that could result in incorrect fee minting. Low-severity issues highlighted flaws like failing USDT approvals and the potential for protocol rebalancing failures due to dust tokens. Minor issues included incorrect event emissions and outdated ownership data in portfolio lists. Overall, the competition demonstrated Hats.finance's model of enhancing DeFi security by leveraging decentralized and competitive auditing methods to identify and resolve critical issues swiftly.

## Disclaimer


This report does not guarantee the absolute security of the audited contracts. Project teams are strongly advised to implement continuous monitoring and verify that deployed contracts match the audited versions. Users should exercise caution and, where possible, confirm contract integrity before interacting with critical smart contracts.

The Velvet Capital audit competition illustrates the collaborative effort in identifying and rectifying potential vulnerabilities, enhancing the overall security and functionality of the platform.


Hats.finance does not provide any guarantee or warranty regarding the security of this project. Smart contract software should be used at the sole risk and responsibility of users.

