# Alchemix Finance Vulnerability Report
Critical Slippage Protection Bypass in WstETH Adapter
This repository contains proof-of-concept code demonstrating a critical vulnerability in Alchemix Finance's WstETH adapter that can result in substantial user losses despite slippage protection settings.

# Vulnerability Summary
The WstETH adapter in Alchemix Finance completely ignores user-specified slippage protection parameters. When a user sets a slippage limit (e.g., 5%) during withdrawals, this limit is never passed to the Curve pool exchange function. Instead, a hardcoded ZERO is used, leaving users with no protection against price manipulation.

This allows attackers to perform sandwich attacks that can cause users to lose up to 33% of their funds despite believing they are protected by their slippage settings.

# Repository Structure
v2-foundry/ - Cloned Alchemix codebase to demonstrate the vulnerability exists in their actual code
v2-foundry/test/AlchemixDirectVulnerabilityTest.sol - The direct test proving the vulnerability with real contract addresses
results/ - Contains terminal outputs, logs, and screenshots verifying the vulnerability

## Proof of Concept
The vulnerability is demonstrated by:

-Connecting to real Alchemix contracts deployed on mainnet
-Verifying that we're interacting with the correct contracts by checking adapter.token()
-Showing the execution flow that causes minAmountOut to be ignored
-Highlighting the exact vulnerable code with the hardcoded zero value

## Attack Steps
1-User calls AlchemistV2.withdraw() with minAmountOut to set a 5% slippage protection
2-The minAmountOut parameter never reaches the Curve pool
3-Attacker can front-run the transaction with large stETH sells
4-User receives significantly less ETH than expected (up to 33% loss)
5-Attacker back-runs to restore price and profits

## How to Run the Test:
1- set your rpc or use my own rpc at line 28 in v2-foundry/test/AlchemixDirectVulnerabilityTest.sol
2- export FOUNDRY_ETH_RPC_URL=https://mainnet.infura.io/v3/YOUR_API_KEY
3- Run the test to verify the vulnerability in just 1 command:
forge test --match-path test/AlchemixDirectVulnerabilityTest.sol -vvv

## Vulnerability Fix
The fix for this vulnerability requires three changes:

1-Update AlchemistV2._unwrap() to pass minAmountOut to adapter.unwrap()
2-Add minAmountOut parameter to WstETHAdapter.unwrap() function signature
3-Pass this value to the Curve pool exchange function instead of using 0
 
## Severity and Impact
Severity: Critical
Impact: High (loss of up to 33% of user funds)
Likelihood: High (actively exploitable by MEV bots)
The issue affects all users withdrawing funds from Alchemix using the WstETH adapter.

Disclosure Timeline
Found: May 19, 2025
Reported to Immunefi: May 19, 2025
Fixed: Pending

Contact
For additional information, please contact [hadimahrouei@gmail.com]
