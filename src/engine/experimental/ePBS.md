# Engine API -- ePBS

Engine API changes introduced in ePBS, based on [Cancun](../cancun.md).

## Table of contents

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->

- [Structures](#structures)
  - [InclusionListV1](#inclusionlistv1)
  - [InclusionListSummaryEntryV1](#inclusionlistsummaryentryv1)
  - [InclusionListStatusV1](#inclusionliststatusv1)
  - [ExecutionPayloadVePBS](#executionpayloadvepbs)
- [Methods](#methods)
  - [`engine_getInclusionListV1`](#engine_getinclusionlistv1)
    - [Request](#request-1)
    - [Response](#response-1)
    - [Specification](#specification-1)
  - [`engine_newInclusionListV1`](#engine_newinclusionlistv1)
    - [Request](#request-2)
    - [Response](#response-2)
    - [Specification](#specification-2)
  - [`engine_newPayloadVePBS`](#engine_newpayloadvepbs)
    - [Request](#request-3)
    - [Response](#response-3)
    - [Specification](#specification-3)


<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Structures

### InclusionListV1
This structure maps onto inclusion list object from [{pending EIP}](). The fields are encoded as follows:
- `transactions`: `Array of DATA` - Array of transaction objects, each object is a byte list (`DATA`) representing `TransactionType || TransactionPayload` or `LegacyTransaction` as defined in [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718)
- `summary`: `Array of InclusionListSummaryEntryV1` - Array of summary entries. Each object is an `OBJECT` containing the fields of a `InclusionListSummaryEntryV1` structure.

### InclusionListSummaryEntryV1
This structure maps onto inclusion list summary entry object from [{pending EIP}](). The fields are encoded as follows:
- `address`: `DATA`, 20 Bytes - the address of the transaction sender.
- `gasLimit`: `QUANTITY`, 64 Bits - the transaction gas limit.

### InclusionListStatusV1
This structure contains the result of processing an inclusion list. The field is encoded as follow:
- `status`: `enum` - `"VALID" | "INVALID"`
- `validationError`: `String|null` - a message providing additional details on the validation error if the inclusion list is classified as `INVALID`.

### ExecutionPayloadVePBS

This structure has the syntax of [`ExecutionPayloadV3`](../cancun.md#executionpayloadv3) and appends the new fields: `inclusionListSummary` and `inclusionListExclusions`.

- `parentHash`: `DATA`, 32 Bytes
- `feeRecipient`:  `DATA`, 20 Bytes
- `stateRoot`: `DATA`, 32 Bytes
- `receiptsRoot`: `DATA`, 32 Bytes
- `logsBloom`: `DATA`, 256 Bytes
- `prevRandao`: `DATA`, 32 Bytes
- `blockNumber`: `QUANTITY`, 64 Bits
- `gasLimit`: `QUANTITY`, 64 Bits
- `gasUsed`: `QUANTITY`, 64 Bits
- `timestamp`: `QUANTITY`, 64 Bits
- `extraData`: `DATA`, 0 to 32 Bytes
- `baseFeePerGas`: `QUANTITY`, 256 Bits
- `blockHash`: `DATA`, 32 Bytes
- `transactions`: `Array of DATA` - Array of transaction objects, each object is a byte list (`DATA`) representing `TransactionType || TransactionPayload` or `LegacyTransaction` as defined in [EIP-2718](https://eips.ethereum.org/EIPS/eip-2718)
- `withdrawals`: `Array of WithdrawalV1` - Array of withdrawals, each object is an `OBJECT` containing the fields of a `WithdrawalV1` structure.
- `blobGasUsed`: `QUANTITY`, 64 Bits
- `excessBlobGas`: `QUANTITY`, 64 Bits
- `inclusionListSummary`: `Array of InclusionListSummaryEntryV1` - Array of summary entries. Each object is an `OBJECT` containing the fields of a `InclusionListSummaryEntryV1` structure.
- `inclusionListExclusions`: `Array of QUANTITY` - Array of transaction indices of parent block to exclude for verification, each object is a `QUANTITY` representing the index.

## Methods

### `engine_getInclusionListV1`

#### Request

* method: `engine_getInclusionListV1`
* params:
  1. `parentHash`: `DATA`, 32 Bytes - hash of the block which the returning inclusion list bases on
* timeout: 1s

#### Response

* result: [`InclusionListV1`](#inclusionlistv1)
* error: code and message set in case an exception happens while getting the inclusion list.

#### Specification
1. Client software **MUST** return the most recent version of the inclusion list based on `parentHash`.
2. Client software **MUST** return `-32602: Invalid parameters` error if the `parentHash` is empty.
3. Client software **MUST** ensure that the `parentHash` exists and corresponds to a valid block. It must return `-38001: Unknown payload` error if it doesn't.

### `engine_newInclusionListV1`
#### Request

* method: `engine_newInclusionListV1`
* params:
  1. `inclusionList`: [`InclusionListV1`](#inclusionlistv1) - The inclusion list to be processed.
  2. `parentHash`: `DATA`, 32 Bytes - hash of the block whose corresponding execution state will be used to validate against the inclusion list.
* timeout: 1s

#### Response

* result: [`InclusionListStatusV1`](#inclusionliststatusv1) 
* error: code and message set in case an exception happens while processing the inclusion list.

#### Specification
1. Client software **MUST** return `-32602: Invalid parameters` error if the `parentHash` is empty.
2. Client software **MUST** ensure that the `parentHash` exists and corresponds to a valid block. It must return `-38001: Unknown payload` error if it doesn't.
3. Client software **MUST** validate the `inclusionList` based on the `parentHash`. It must respond the call in the following way:
    * `{status: INVALID}` if `inclusionList` validation has failed.
    * `{status: VALID}` if `inclusionList` validation has succeeded.
4. Client software **MUST** specify the error in the response if the validation of `inclusionList` fails.
5. If any of the above fails due to errors unrelated to the normal processing flow of the method, client software **MUST** respond with an error object.

### `engine_newPayloadVePBS`
#### Request

* method: `engine_newPayloadVePBS`
* params:
  1. `executionPayload`: [`ExecutionPayloadVePBS`](#executionpayloadvepbs).
  2. `expectedBlobVersionedHashes`: `Array of DATA`, 32 Bytes - Array of expected blob versioned hashes to validate.
  3. `parentBeaconBlockRoot`: `DATA`, 32 Bytes - Root of the parent beacon block.
* timeout: 8s

#### Response

Refer to the response for [`engine_newPayloadV2`](../shanghai.md#engine_newpayloadv2).

#### Specification

This method follows the same specification as [`engine_newPayloadV3`](../shanghai.md#engine_newpayloadv2) with the addition of the following:

1. Client software **MUST** return `-38005: Unsupported fork` error if the `timestamp` of the payload does not fall within the time frame of the ePBS fork.
2. Client software **MUST** perform inclusion list verification before state transition given the additional fields. The validation **MUST** check the following things:
    1. Verify if the provided `inclusionListSummary` contains at least one entry for each transaction in parent block identified by the index in the `inclusionListExclusions` array in strictly increasing order.
    2. Verify if the remaining entries in `inclusionListSummary` satisfies the top entries of the payload. 
    3. Return appropriate error in block validation if the above verification fails.
