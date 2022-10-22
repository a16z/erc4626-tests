# ERC4626 Property Tests

Foundry (dapptools-style) property-based tests for [ERC4626] standard conformance.

[ERC4626]: <https://eips.ethereum.org/EIPS/eip-4626>

You can read our post on "_[Executable ERC4626 Standard Properties]_."

[Executable ERC4626 Standard Properties]: TBA

## Overview

#### What is it?
- Test suites for checking if the given ERC4626 implementation satisfies the **standard requirements**.
- Dapptools-style **property-based tests** for fuzzing or symbolic execution testing.
- Tests that are **independent** from implementation details, thus applicable for any ERC4626 vaults.

#### What isn’t it?
- It does NOT test implementation-specific details, e.g., how to generate and distribute yields, how to compute the share price, etc.

#### Testing properties:

- **Round-trip properties**: no one can make a free profit by depositing and immediately withdrawing back and forth.[^rt]

[^rt]: See [here](https://hackmd.io/@daejunpark/BJMGbD435#Invariants) for formal analysis.

- **Functional correctness**: the `deposit()`, `mint()`, `withdraw()`, and `redeem()` functions update the balance and allowance properly.

- The `preview{Deposit,Redeem}()` functions **MUST NOT over-estimate** the exact amount.[^1]

[^1]: That is, the `deposit()` and `redeem()` functions “MUST return the same or more amounts as their preview function if called in the same transaction.”

- The `preview{Mint,Withdraw}()` functions **MUST NOT under-estimate** the exact amount.[^2]

[^2]: That is, the `mint()` and `withdraw()` functions “MUST return the same or fewer amounts as their preview function if called in the same transaction.”

- The `convertTo{Shares,Assets}` functions “**MUST NOT show any variations** depending on the caller.”

- The `asset()`, `totalAssets()`, and `max{Deposit,Mint,Withdraw,Redeem}()` functions “**MUST NOT revert**.”

## Usage

**Step 0**: Install [foundry] and add [forge-std] in your vault repo:
```bash
$ curl -L https://foundry.paradigm.xyz | bash

$ cd /path/to/your-erc4626-vault
$ forge install foundry-rs/forge-std
```

[foundry]: <https://getfoundry.sh/>
[forge-std]: <https://github.com/foundry-rs/forge-std>

**Step 1**: Add this [erc4626-tests] as a dependency to your vault:
```bash
$ cd /path/to/your-erc4626-vault
$ forge install a16z/erc4626-tests
```

[erc4626-tests]: <https://github.com/a16z/erc4626-tests>

**Step 2**: Extend the abstract test contract [`ERC4626.test.sol`](ERC4626.test.sol) with your own custom vault setup method, for example:

```solidity
// SPDX-License-Identifier: AGPL-3.0
pragma solidity >=0.8.0 <0.9.0;

import "erc4626-tests/ERC4626.test.sol";

import { ERC20Mock   } from "/path/to/mocks/ERC20Mock.sol";
import { ERC4626Mock } from "/path/to/mocks/ERC4626Mock.sol";

contract ERC4626StdTest is ERC4626Test {

    function setUp() public override {
        __underlying__ = address(new ERC20Mock("Mock ERC20", "MERC20", 18));
        __vault__ = address(new ERC4626Mock(ERC20Mock(__underlying__), "Mock ERC4626", "MERC4626"));
        __delta__ = 0;  // the size rounding errors to be tolerated in checking test results
    }

}
```

**Step 3**: Run `forge test`

```
$ forge test
```

## Examples

Below are examples of adding these property tests to existing ERC4626 vaults:
- [OpenZeppelin ERC4626] [[diff](https://github.com/daejunpark/openzeppelin-contracts/commit/c4a495447cb7345c29b25bda3b5365276bb2f29b)]
- [Solmate ERC4626] [[diff](https://github.com/daejunpark/solmate/commit/36dc4adcf035b1e94ffe795d21a4c6e513ddcfbc)]
- [Revenue Distribution Token] [[diff](https://github.com/daejunpark/revenue-distribution-token/commit/7ca627002ceb3af02c727fe5dda9a1170adf7a6d)]
- [Yield Daddy ERC4626 wrappers] [[diff](https://github.com/daejunpark/yield-daddy/commit/15a76913f082cd98fdad47d1a5f7932115a59c36)][^bug]

[OpenZeppelin ERC4626]: <https://github.com/OpenZeppelin/openzeppelin-contracts/blob/a1948250ab8c441f6d327a65754cb20d2b1b4554/contracts/token/ERC20/extensions/ERC4626.sol>
[Solmate ERC4626]: <https://github.com/transmissions11/solmate/blob/c2594bf4635ad773a8f4763e20b7e79582e41535/src/mixins/ERC4626.sol>
[Revenue Distribution Token]: <https://github.com/maple-labs/revenue-distribution-token/blob/be9592fd72bfa7142a217507f2d5500a7856329e/contracts/RevenueDistributionToken.sol>
[Yield Daddy ERC4626 wrappers]: <https://github.com/timeless-fi/yield-daddy>

[^bug]: Our property tests indeed revealed a [bug](https://github.com/timeless-fi/yield-daddy/issues/7) in their eToken testing mock contract. The tests passed after it is [fixed](https://github.com/daejunpark/yield-daddy/commit/721cf4bd766805fd409455434aa5fd1a9b2df25c).


## Disclaimer

_These smart contracts are being provided as is. No guarantee, representation or warranty is being made, express or implied, as to the safety or correctness of the user interface or the smart contracts. They have not been audited and as such there can be no assurance they will work as intended, and users may experience delays, failures, errors, omissions or loss of transmitted information. THE SMART CONTRACTS CONTAINED HEREIN ARE FURNISHED AS IS, WHERE IS, WITH ALL FAULTS AND WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING ANY WARRANTY OF MERCHANTABILITY, NON- INFRINGEMENT OR FITNESS FOR ANY PARTICULAR PURPOSE. Further, use of any of these smart contracts may be restricted or prohibited under applicable law, including securities laws, and it is therefore strongly advised for you to contact a reputable attorney in any jurisdiction where these smart contracts may be accessible for any questions or concerns with respect thereto. Further, no information provided in this repo should be construed as investment advice or legal advice for any particular facts or circumstances, and is not meant to replace competent counsel. a16z is not liable for any use of the foregoing, and users should proceed with caution and use at their own risk. See a16z.com/disclosures for more info._
