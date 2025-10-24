SmartFolio
==========

Smart Portfolio Manager: Decentralized Asset Allocation and Risk Management
---------------------------------------------------------------------------

SmartFolio is a **Clarity smart contract** designed to implement a decentralized, automated portfolio management system. It provides comprehensive functionalities for asset allocation, dynamic rebalancing, and robust risk management across multiple investment strategies. Built on the Stacks blockchain, this contract ensures secure fund management, transparent access controls, and a full audit trail for all portfolio actions.

* * * * *

🚀 Features
-----------

-   **Decentralized Portfolio Creation:** Users can create and own their investment portfolios, tailored to specific strategies.

-   **Dynamic Asset Allocation:** Supports various asset types and implements advanced, dynamic algorithms for asset allocation, demonstrated by the `optimize-portfolio-dynamic` function.

-   **Automated Rebalancing:** Includes mechanisms to check for and execute portfolio rebalancing when holdings drift from target allocations, utilizing a configurable `MIN-REBALANCE-THRESHOLD`.

-   **Comprehensive Risk Management:** Enforces strict risk constraints, including a `MAX-SINGLE-ASSET-WEIGHT` limit (50%) and strategy-specific maximum drawdown (`max-drawdown`).

-   **Strategy Configuration:** Allows for the definition and customization of multiple investment strategies with different risk profiles and rebalancing frequencies.

-   **Secure Access Controls:** Utilizes a detailed `user-permissions` map to manage administrative, asset management, and portfolio creation rights.

-   **Fee Structure:** Defines clear `MANAGEMENT-FEE-BP` (2%) and `PERFORMANCE-FEE-BP` (2%) for service sustainability, tracked via `fee-accounts`.

* * * * *

🛠 Contract Details
-------------------

### Constants

The contract is configured with several constants to establish system-wide limits and financial parameters:

| **Constant** | **Value** | **Description** |
| --- | --- | --- |
| `CONTRACT-OWNER` | `tx-sender` | The principal who deploys the contract. |
| `ADMIN-ROLE` | `as-contract tx-sender` | A principal identifier used for admin actions. |
| `MAX-ASSETS` | `u10` | Maximum number of different assets allowed per portfolio. |
| `MAX-STRATEGIES` | `u5` | Maximum number of investment strategies supported. |
| `MANAGEMENT-FEE-BP` | `u200` | Annual management fee (200 basis points = 2%). |
| `PERFORMANCE-FEE-BP` | `u200` | Performance fee (200 basis points = 2%). |
| `MAX-SINGLE-ASSET-WEIGHT` | `u5000` | Maximum percentage (in basis points) of a portfolio that can be allocated to a single asset (50%). |
| `MIN-REBALANCE-THRESHOLD` | `u500` | Minimum deviation (in basis points) required from the target allocation to trigger a rebalance check (5%). |

### Data Structures

The core data management of the system relies on a set of data variables and maps:

| **Data Structure** | **Type** | **Key/Fields** | **Description** |
| --- | --- | --- | --- |
| `portfolio-counter` | `uint` | - | Tracks the total number of portfolios created. |
| `asset-registry` | `map` | `asset-id: uint` | Stores details for each supported token/asset: symbol, decimals, contract address, and active status. |
| `portfolio-holdings` | `map` | `{portfolio-id: uint, asset-id: uint}` | Records the current amount of a specific asset held by a portfolio. |
| `allocation-targets` | `map` | `{portfolio-id: uint, asset-id: uint}` | Stores the target allocation percentage (in basis points) for each asset within a portfolio. |
| `portfolio-metadata` | `map` | `portfolio-id: uint` | Contains high-level portfolio information: owner, name, strategy ID, total value, last rebalance timestamp, and active status. |
| `strategies` | `map` | `strategy-id: uint` | Defines the configuration for each investment strategy: name, risk level (1-5), rebalance frequency (in days), max drawdown, and active status. |
| `user-permissions` | `map` | `principal` | Manages permissions for users: `can-create-portfolio`, `can-manage-assets`, `can-execute-rebalance`, and `is-admin`. |
| `fee-accounts` | `map` | `principal` | Tracks accumulated fees for various fee recipients. |

* * * * *

📝 Functions
------------

### Private Functions

Private functions handle internal logic, validation, and complex calculations. Note that some are currently simplified (`has-permission`, `calculate-portfolio-value`, `needs-rebalancing`) and require integration with external price feeds and more sophisticated logic in a production environment.

-   `(validate-allocation (portfolio-id uint) (total-percentage uint))`

    -   **Purpose:** Ensures the sum of all target asset allocations for a portfolio is within an acceptable range (95% to 100% of basis points).

    -   **Errors:** `u1001` (Max 100%), `u1002` (Min 95%).

-   `(has-permission (user principal) (permission (string-ascii 32)))`

    -   **Purpose:** Checks if a given user principal holds a specific administrative or management permission. *Currently simplified to only check for `CONTRACT-OWNER`.*

    -   **Return:** `(ok (is-eq user CONTRACT-OWNER))`.

-   `(calculate-portfolio-value (portfolio-id uint))`

    -   **Purpose:** Calculates the real-time total value of a portfolio. *Currently simplified to return `u0`.*

-   `(needs-rebalancing (portfolio-id uint))`

    -   **Purpose:** Determines if a portfolio's current holdings have deviated enough from their targets to necessitate a rebalance. *Currently simplified to return `(ok true)`.*

### Public Functions

Public functions expose the core functionality of the portfolio manager to users and other contracts.

#### Core Management

-   `(initialize-contract)`

    -   **Purpose:** Initializes the contract by setting the default, full permissions for the `CONTRACT-OWNER`.

    -   **Pre-conditions:** Only callable by the `CONTRACT-OWNER` (checked with `u1005`).

-   `(add-asset (asset-id uint) (symbol (string-utf8 12)) (decimals uint) (contract-address principal))`

    -   **Purpose:** Registers a new asset (token contract) into the `asset-registry`.

    -   **Pre-conditions:** Requires `"manage-assets"` permission and the `asset-id` must be less than `MAX-ASSETS`.

-   `(create-strategy (strategy-id uint) (name (string-utf8 32)) (risk-level uint) (rebalance-frequency uint) (max-drawdown uint))`

    -   **Purpose:** Defines a new investment strategy configuration.

    -   **Pre-conditions:** Requires `"admin"` permission, `strategy-id` less than `MAX-STRATEGIES`, and `risk-level` between `u1` and `u5`.

#### Portfolio Operations

-   `(create-portfolio (name (string-utf8 64)) (strategy-id uint))`

    -   **Purpose:** Creates a new portfolio, increments the `portfolio-counter`, and stores the metadata.

    -   **Pre-conditions:** Requires `"create-portfolio"` permission.

    -   **Post-conditions:** Returns the new `portfolio-id`.

-   `(set-allocation-targets (portfolio-id uint) (asset-id uint) (target-percentage uint))`

    -   **Purpose:** Sets the desired allocation percentage (in basis points) for a specific asset within a portfolio.

    -   **Pre-conditions:** Requires `CONTRACT-OWNER` status (checked with `u1013`) and `target-percentage` must not exceed `MAX-SINGLE-ASSET-WEIGHT` (5000 basis points).

-   `(execute-rebalance (portfolio-id uint))`

    -   **Purpose:** Triggers the process of buying/selling assets to bring the portfolio holdings back to the target allocations. *Currently simplified.*

    -   **Pre-conditions:** Requires `CONTRACT-OWNER` status (checked with `u1016`).

#### Advanced Optimization

-   `(optimize-portfolio-dynamic (portfolio-id uint) (risk-tolerance uint) (time-horizon uint) (volatility-factor uint))`

    -   **Purpose:** Implements a sophisticated, **Modern Portfolio Theory (MPT)-inspired** algorithm to calculate *new* risk-adjusted asset allocation targets.

    -   **Logic:** It computes an `optimization-factor` based on user inputs:

        -   **Risk Tolerance:** Adjusts up for higher tolerance ($\ge 3 \rightarrow 110\%$) and down for lower tolerance ($\rightarrow 90\%$).

        -   **Time Horizon:** Adjusts up for longer horizons ($\ge 365 \text{ days} \rightarrow 105\%$) and down for shorter ones ($\rightarrow 95\%$).

        -   **Volatility Factor:** Adjusts down for higher volatility ($\ge 5000 \text{BP} \rightarrow 80\%$) and up for lower volatility ($\rightarrow 120\%$).

    -   The `optimization-factor` is then applied to base target allocations for assets u1, u2, and u3, constrained by the `MAX-SINGLE-ASSET-WEIGHT` and a minimum target of `u100` (1%).

    -   **Pre-conditions:** Requires `CONTRACT-OWNER` status (checked with `u1020`).

    -   **Errors:** `u1025` if the total optimized allocation is less than the 95% threshold.

* * * * *

🚦 Error Codes
--------------

The contract utilizes custom error codes (starting from `u1000`) for clear failure reporting:

| **Error Code** | **Description** |
| --- | --- |
| `u1001` | Allocation validation failed: Total percentage exceeds 100%. |
| `u1002` | Allocation validation failed: Total percentage is below 95%. |
| `u1005` | Initialization failed: Only the contract owner can call. |
| `u1006` | Permission denied: User lacks required asset management rights. |
| `u1007` | Asset limit exceeded: `asset-id` is $\ge$ `MAX-ASSETS`. |
| `u1008` | Permission denied: User lacks required admin rights. |
| `u1009` | Strategy limit exceeded: `strategy-id` is $\ge$ `MAX-STRATEGIES`. |
| `u1010` | Invalid risk level: Must be between 1 and 5. |
| `u1011` | Permission denied: User lacks required portfolio creation rights. |
| `u1013` | Unauthorized access: Only the contract owner can set allocation targets. |
| `u1015` | Risk violation: Asset target exceeds `MAX-SINGLE-ASSET-WEIGHT`. |
| `u1016` | Unauthorized access: Only the contract owner can execute rebalance. |
| `u1020` | Unauthorized access: Only the contract owner can call dynamic optimization. |
| `u1025` | Invalid optimization result: Total optimized allocation is below 95%. |

* * * * *

🤝 Contribution
---------------

SmartFolio is an open-source project and welcomes contributions from the community. If you have suggestions for new features, improvements to the optimization algorithms, or bug fixes, please follow these guidelines:

1.  **Fork** the repository and clone it locally.

2.  Create a new, descriptive **branch** for your feature or fix (e.g., `feature/add-price-oracle` or `fix/rebalance-logic`).

3.  Ensure your code adheres to the existing **Clarity style and conventions**.

4.  Write **unit tests** for all new functionality or changes to existing logic.

5.  Create a **Pull Request (PR)** against the `main` branch with a clear description of your changes and why they are necessary.

**Areas for Future Development:**

-   **Price Oracle Integration:** Implement a robust mechanism to fetch real-time, tamper-proof asset prices to correctly calculate `total-value` and execute trades.

-   **Trading Interface:** Develop public functions that interact with a decentralized exchange (DEX) contract to perform actual buy/sell operations during rebalancing.

-   **Decentralized Permissions:** Move beyond the simplified `CONTRACT-OWNER` check in `has-permission` to a decentralized governance model or multi-sig.

-   **Strategy Complexity:** Enhance the `strategies` map with more granular risk and performance parameters.

-   **Fee Collection:** Implement the fee collection logic using the `MANAGEMENT-FEE-BP` and `PERFORMANCE-FEE-BP`.

* * * * *

📜 License
----------

The SmartFolio contract is released under the **MIT License**.

```
MIT License

Copyright (c) 2025 SmartFolio Developers

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
