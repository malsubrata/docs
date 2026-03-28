# TeraWallet Technical Reference & Project Context

This document serves as the primary instructional context for the TeraWallet project, containing deep-dive technical logic, architectural patterns, and workflow details extracted from the codebase.

## 🏗 Architecture Overview

TeraWallet follows a modular, object-oriented architecture built on top of WordPress and WooCommerce.

- **Centralized Ledger**: All balance operations are recorded in a custom database table (`woo_wallet_transactions`). The balance is calculated dynamically using SQL `SUM()` to ensure 100% accuracy.
- **Concurrency Protection**: The system employs **MySQL-level locking** (`GET_LOCK`) during ledger writes to prevent race conditions in high-concurrency environments (e.g., multiple simultaneous AJAX requests).
- **Singleton Lifecycle**: The `Woo_Wallet` class manages sub-modules (Wallet, Cashback, API, Settings) and handles plugin-wide hooks.
- **Hook-Driven**: Extremely extensible via filters and actions, allowing seamless integration with multi-vendor marketplaces like Dokan, WCFM, and WCMarketplace.

## 💰 Core Wallet Logic (`Woo_Wallet_Wallet`)

- **Balance Calculation**: Performed via `get_wallet_balance()`. It sums credits and subtracts debits for a specific user ID.
- **Atomic Operations**: `credit()` and `debit()` are the only two entry points for balance modification. They both use the locking mechanism.
- **Metadata**: Transactions support custom metadata stored in `woo_wallet_transaction_meta`, allowing for detailed ledger entries (e.g., "Cashback for Order #123").

## 🎁 Cashback Engine (`Woo_Wallet_Cashback`)

- **Rules**: Supports three primary calculation modes:
    1. **Cart-wise**: Based on the total order value.
    2. **Product-wise**: Individual products define their own reward amounts/types.
    3. **Category-wise**: Rewards based on product taxonomy.
- **Coupons as Cashback**: A unique feature where standard WooCommerce coupons can be "converted" into wallet rewards instead of direct discounts.
- **Lifecycle**: Calculated during checkout -> Stored in order meta -> Credited when order status hits the "Cashback Status" (configurable).

## 💳 Payment Gateway & Partial Payments

- **Wallet Gateway**: Implements `WC_Payment_Gateway`. It is only available if the user has a sufficient balance for the *entire* order (unless partial payment is on).
- **Partial Payment Implementation**: 
    - Works by adding a **negative fee** (`_via_wallet_partial_payment`) to the cart.
    - This effectively reduces the "Total" WooCommerce needs to collect from other gateways.
    - The partial amount is debited from the wallet during the `woocommerce_order_processed` hook.
- **Refunds**: Fully supports native WooCommerce refunds. Refunding an order paid via wallet automatically triggers a `credit()` transaction back to the customer's ledger.

## 🔄 Core Flows

### Top-up Flow
1. User purchases a "Wallet Topup" product.
2. Order completion triggers `wallet_credit_purchase()`.
3. Funds are credited to the ledger.

### Checkout Flow (Full Payment)
1. User selects "Wallet" gateway.
2. `process_payment()` checks for sufficient funds.
3. `woocommerce_pre_payment_complete()` triggers the `debit()`.
4. Order is marked as paid.

### Checkout Flow (Partial Payment)
1. User applies wallet balance at checkout.
2. Negative fee is added to cart.
3. User pays the remaining balance via another gateway (e.g., Stripe).
4. `woocommerce_order_processed()` triggers the `debit()` for the wallet portion.

## 📂 Key Files for Documentation
- `woo-wallet-src/includes/class-woo-wallet-wallet.php`: Balance & Locking logic.
- `woo-wallet-src/includes/class-woo-wallet-cashback.php`: Reward calculation logic.
- `woo-wallet-src/includes/class-woo-wallet-payment-method.php`: Gateway integration.
- `woo-wallet-src/includes/helper/woo-wallet-util.php`: Global utility functions.
