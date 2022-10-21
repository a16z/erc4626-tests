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

More examples of adding these property tests to existing ERC4626 vaults can be found [here](https://github.com/daejunpark/erc4626-tests-demo).

## Disclaimer

_These smart contracts are being provided as is. No guarantee, representation or warranty is being made, express or implied, as to the safety or correctness of the user interface or the smart contracts. They have not been audited and as such there can be no assurance they will work as intended, and users may experience delays, failures, errors, omissions or loss of transmitted information. THE SMART CONTRACTS CONTAINED HEREIN ARE FURNISHED AS IS, WHERE IS, WITH ALL FAULTS AND WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING ANY WARRANTY OF MERCHANTABILITY, NON- INFRINGEMENT OR FITNESS FOR ANY PARTICULAR PURPOSE. Further, use of any of these smart contracts may be restricted or prohibited under applicable law, including securities laws, and it is therefore strongly advised for you to contact a reputable attorney in any jurisdiction where these smart contracts may be accessible for any questions or concerns with respect thereto. Further, no information provided in this repo should be construed as investment advice or legal advice for any particular facts or circumstances, and is not meant to replace competent counsel. a16z is not liable for any use of the foregoing, and users should proceed with caution and use at their own risk. See a16z.com/disclosures for more info._
