
# KYCKey - Decentralized KYC Verification Platform

A decentralized smart contract on the Stacks blockchain to manage KYC (Know Your Customer) verification status for user addresses. This platform allows users to request KYC verification, and the contract owner to approve, reject, revoke verification, and transfer contract ownership securely.

---

## Table of Contents

* [Overview](#overview)
* [Features](#features)
* [Status Codes](#status-codes)
* [Error Codes](#error-codes)
* [Contract Variables and Data Structures](#contract-variables-and-data-structures)
* [Functions](#functions)

  * [Public Functions](#public-functions)
  * [Read-Only Functions](#read-only-functions)
* [Usage](#usage)
* [Security Considerations](#security-considerations)
* [License](#license)

---

## Overview

KYCrypt provides a decentralized way to manage KYC verification status on-chain by associating status and KYC data with blockchain addresses. The contract owner controls the verification process, including approving or rejecting verification requests.

---

## Features

* Users can **request KYC verification** by submitting their KYC data.
* Contract owner can **verify** or **reject** these requests.
* Contract owner can **revoke** verification for an address.
* Ownership of the contract can be transferred securely to another principal.
* Verification status and KYC data are stored on-chain and publicly accessible.
* Input validation for addresses and KYC data to prevent misuse.
* Supports multiple verification statuses for tracking request lifecycle.

---

## Status Codes

| Status Code | Meaning      |
| ----------- | ------------ |
| `0`         | Not Verified |
| `1`         | Pending      |
| `2`         | Verified     |
| `3`         | Rejected     |

---

## Error Codes

| Code                          | Description                         |
| ----------------------------- | ----------------------------------- |
| `ERR_UNAUTHORIZED (100)`      | Unauthorized action (not owner)     |
| `ERR_ALREADY_VERIFIED (101)`  | Address already verified or pending |
| `ERR_NOT_VERIFIED (102)`      | Address is not verified             |
| `ERR_INVALID_STATUS (103)`    | Invalid status for operation        |
| `ERR_INVALID_INPUT (104)`     | Invalid input data or address       |
| `ERR_INVALID_NEW_OWNER (105)` | New owner address invalid           |

---

## Contract Variables and Data Structures

* **contract-owner**: The principal (address) of the current contract owner.
* **verified-addresses**: Map storing verification info per address:

  * `status`: uint (see Status Codes)
  * `timestamp`: uint (block height of status change)
  * `kyc-data`: string (up to 500 UTF-8 chars)
  * `verifier`: principal (who verified or submitted)

---

## Functions

### Public Functions

* **request-verification(kyc-data: string-utf8 500) → (response bool)**

  * Called by any user to request KYC verification.
  * Input: KYC data string.
  * Status must be `0` (Not Verified).
  * Sets status to `1` (Pending).

* **verify-address(address: principal) → (response bool)**

  * Only contract owner can call.
  * Verifies a pending address (`status == 1`).
  * Changes status to `2` (Verified).

* **reject-verification(address: principal) → (response bool)**

  * Only contract owner can call.
  * Rejects a pending address (`status == 1`).
  * Changes status to `3` (Rejected).

* **revoke-verification(address: principal) → (response bool)**

  * Only contract owner can call.
  * Revokes verification from `Pending (1)` or `Verified (2)` statuses.
  * Changes status back to `0` (Not Verified).

* **transfer-ownership(new-owner: principal) → (response bool)**

  * Only current contract owner can call.
  * Transfers ownership to a new valid principal.
  * Prevents setting owner to same address or invalid principals.

### Read-Only Functions

* **get-verification-status(address: principal) → (tuple)**

  * Returns the verification record for an address.
  * Defaults to `{status: 0, timestamp: 0, kyc-data: "", verifier: tx-sender}` if none.

* **is-contract-owner(address: principal) → (bool)**

  * Checks if the given address is the contract owner.

---

## Usage

1. **Deploy the contract:** The deployer is set as the initial contract owner.
2. **Users request verification:** Call `request-verification` with valid KYC data.
3. **Contract owner approves/rejects requests:**

   * Call `verify-address` to approve.
   * Call `reject-verification` to reject.
4. **Owner can revoke verifications or transfer contract ownership.**
5. **Anyone can query verification status with `get-verification-status`.**

---

## Security Considerations

* Only the contract owner can verify, reject, revoke verification, or transfer ownership.
* Users cannot manipulate verification status of other addresses.
* Contract prevents self-interaction and invalid principal inputs.
* KYC data length limited to 500 UTF-8 characters to avoid storage abuse.
* Ownership transfer prevents setting the same owner again.
* Status changes validated to maintain correct state transitions.

---

## License

This smart contract code is provided as-is without warranty. Use at your own risk. You may modify and distribute under terms compatible with your project needs.

---