# Movement Report API

> **Version:** 1.1  
> **Date:** September 2026  
> **Base URL:** `https://www.sagepayments.net/web_services/Reconciliation`

The Movement Report API provides access to merchant deposit and transaction reporting data. This endpoint will work for merchant accounts boarded on either Paya Core (Sage Exchange/Sage Payments) or NIPG Connect (Paya Connect).

## Contents

- [Authentication and Headers](#authentication-and-headers)
- [Deposits API](#deposits-api)
- [Transactions API](#transactions-api)
- [Error responses](#error-responses)
- [Code examples](#code-examples)

## Authentication and headers

### Request Headers

| Header | Required | Description | Example |
|---|---:|---|---|
| `Merchant-Auth` | Yes | Merchant authentication credentials encoded as a JSON string. For PCON merchants, include `DeveloperId` in the same JSON object. | `{"UserId":"USER_ID","UserKey":"USER_KEY","MerchantType":"CoreGateway"}` |
| `x-correlationid` | Preferred | GUID used to track the request end to end. When supplied, the same value is returned in the response. | `123e4567-e89b-12d3-a456-426614174000` |
| `Content-Type` | Yes | Request media type. | `application/json` |
| `Accept` | Yes | Expected response media type. | `application/json` |

### `Merchant-Auth` header format

The `Merchant-Auth` header must contain a JSON string with the following fields.

| Field | Required | Description |
|---|---:|---|
| `UserId` | Yes | API user identifier. Core Gateway users provide their 12-digit `GatewayId`or VT ID. PCON users provide their `User-Id`. This is not the Merchant ID. |
| `UserKey` | Yes | Core Gateway users provide their `GatewayKey` or Merchant Key. PCON users provide their `User-API-Key`. |
| `MerchantType` | Yes | Use `CoreGateway` for Core Gateway users or `PCON` for PCON users. |
| `DeveloperId` | PCON only | Developer ID assigned to the integrator. Core Gateway users omit this field. |

#### Core Gateway example

```json
{
  "UserId": "GATEWAY_ID",
  "UserKey": "GATEWAY_KEY",
  "MerchantType": "CoreGateway"
}
```

#### PCON example

```json
{
  "UserId": "USER_ID",
  "UserKey": "USER_API_KEY",
  "MerchantType": "PCON",
  "DeveloperId": "DEVELOPER_ID"
}
```

> [!CAUTION]
> Do not commit real user IDs, API keys, gateway keys, or developer IDs to source control. Use environment variables or a secret manager.

### Response headers

| Header | Description |
|---|---|
| `x-correlationid` | Request tracking identifier. If the request supplies a correlation ID, the response returns the same value. Otherwise, the API returns a new GUID. |
| `Content-Type` | Response media type: `application/json`. |

## Deposits API

### Get deposits

Retrieves deposit information for a merchant account and date range.

```http
POST /v1/deposits
```

### Request body

```json
{
  "MerchantAccountNumber": "3948123412341234",
  "DateFrom": "09012026",
  "DateTo": "09142026"
}
```

### Request schema

| Field | Type | Required | Max length | Format | Description |
|---|---|---:|---:|---|---|
| `MerchantAccountNumber` | string | Yes | 16 | Numeric | 16-digit merchant account number. |
| `DateFrom` | string | Yes | 8 | `MMDDYYYY` | Start date for the deposit search. Must not be more than 90 days in the past or in the future. |
| `DateTo` | string | Yes | 8 | `MMDDYYYY` | End date for the deposit search. Must not be more than 90 days in the past or in the future. |

### Validation rules

- `DateFrom` and `DateTo` must use `MMDDYYYY`, for example `09082026` for September 8, 2026.
- Dates cannot be more than 90 days in the past.
- Dates cannot be in the future.
- `DateFrom` must be less than or equal to `DateTo`.
- The requested date range cannot exceed 31 days.
- `MerchantAccountNumber` must contain exactly 16 numeric digits.

### Successful response

The endpoint returns an array of deposit objects.

```json
[
  {
    "ACHAttribute": "C",
    "ACHAttributeDescription": "Combine",
    "Amount": "15750.50",
    "CardType": "V",
    "CardTypeDesc": "Visa",
    "ChargebackCaseNumber": "",
    "DebitCreditIndicator": "C",
    "DebitCreditIndicatorDesc": "Credit",
    "EffectiveDate": "09082026",
    "MID": "3948123412341234",
    "NetACHAmount": "15325.75",
    "ProcessingDate": "09092026",
    "RecordType": "Total",
    "ReferenceNumber": "REF123456",
    "TransactionTypes": "SALE"
  }
]
```

### Deposit object

| Field | Type | Description |
|---|---|---|
| `ACHAttribute` | string | ACH attribute indicator. Possible values: `C`, `D`, `N`, `S`. |
| `ACHAttributeDescription` | string | Deposit handling description: `C` = Combine, `D` = Deposit, `N` = No ACH, `S` = Separate. |
| `Amount` | string | Deposit amount represented as a 17-digit numeric value in the source workbook. |
| `CardType` | string | Card scheme type. |
| `CardTypeDesc` | string | Card type description. |
| `ChargebackCaseNumber` | string | Chargeback case number, when applicable. |
| `DebitCreditIndicator` | string | Debit or credit indicator. |
| `DebitCreditIndicatorDesc` | string | Debit or credit indicator description. |
| `EffectiveDate` | string | Deposit or ACH effective date in `MMDDYYYY` format. |
| `MID` | string | 16-digit merchant identifier associated with the deposit record. |
| `NetACHAmount` | string | Net ACH amount by batch and card plan. |
| `ProcessingDate` | string | Deposit or ACH processing date in `MMDDYYYY` format. |
| `RecordType` | string | Record type identifier. |
| `ReferenceNumber` | string | Reference number associated with the deposit record. |
| `TransactionTypes` | string | Transaction type related to merchant funding. |

## Transactions API

### Get transactions

Retrieves transaction details for a merchant account and date range.

```http
POST /v1/transactions
```

### Request body

```json
{
  "MerchantAccountNumber": "3948123412341234",
  "DateFrom": "09012026",
  "DateTo": "09142026",
  "PageNumber": 1,
  "PageSize": 3000
}
```

### Request schema

| Field | Type | Required | Constraints | Description |
|---|---|---:|---|---|
| `MerchantAccountNumber` | string | Yes | 16 numeric digits | Merchant account number. |
| `DateFrom` | string | Yes | 8 characters, `MMDDYYYY` | Start date for the transaction search. |
| `DateTo` | string | Yes | 8 characters, `MMDDYYYY` | End date for the transaction search. |
| `PageNumber` | integer | No | `>= 1` | Page number. Default: `1`. |
| `PageSize` | integer | No | `1` to `3000` | Transactions per page. Default and maximum: `3000`. |

The same date and merchant account validation rules documented for the Deposits API apply to this endpoint.

### Successful response

```json
{
  "Pagination": {
    "PageNumber": 1,
    "PageSize": 3000,
    "TotalRecords": 12500,
    "TotalPages": 5,
    "HasPreviousPage": false,
    "HasNextPage": true
  },
  "BatchHeaders": [
    {
      "ACHPostDate": "09082026",
      "BatchDate": "09082026",
      "BatchId": "BATCH123",
      "DebitCreditIndicator": "D",
      "DebitCreditIndicatorDesc": "Debit",
      "MerchantId": "3948123412341234",
      "MerchantReferenceNumber": "MRN123456",
      "NetDeposit": "9485.25",
      "RecordType": "BH",
      "RejectReasonCode": "",
      "Transactions": [
        {
          "ACHFlag": "Y",
          "AmexSENumber": "SE12345",
          "AuthAmount": "436.60",
          "AuthCurrencyCode": "USD",
          "AuthCurrencyDecode": "US Dollar",
          "AuthNumber": "AUTH123",
          "BanknetReferenceNumber": "BNK123456",
          "CardBrandFeeCode": "1",
          "CardBrandFeeDesc": "Mastercard Cross Border Domestic Currency",
          "CardholderAccountNumber": "****1234",
          "CardTypeID": "V",
          "CardTypeDesc": "Visa",
          "CEDPIndicator": "N",
          "CommercialCardIndicator": "A",
          "CommercialCardIndicatorDesc": "GSA IGOTS",
          "CrossBorderFeeAmount": "8.75",
          "DBAName": "Merchant DBA Name",
          "DebitCreditIndicator": "D",
          "DebitCreditIndicatorDesc": "Debit",
          "ExtensionRecordIndicator": "N",
          "ForeignExchangeFlag": "N",
          "GatewayReferenceId": "GW123456789",
          "IASFFeeAmount": "3.50",
          "IASFFeeDebitCreditIndicator": "D",
          "IASFFeeType": "IASF01",
          "InterchangeFeeAmount": "12.25",
          "InterchangeFeePercentRate": "1.65",
          "InterchangePerItemRate": "0.10",
          "MCCashBackFee": "2.00",
          "MCCashBackFeeSign": "+",
          "MerchantAccountNumber": "3948123412341234",
          "MerchantReferenceNumber": "MRN123456",
          "NetDepositAmount": "410.00",
          "NetDepositAdjustmentAmount": "15.50",
          "NetDepositAdjustmentDebitCredit": "C",
          "NetworkDebitIdentifier": "NDI001",
          "OriginalTransactionAmount": "436.60",
          "ProductID": "A",
          "ProductIDDesc": "Visa Traditional",
          "PurchaseID": "PURCH123",
          "RecordType": "D",
          "ReferenceNumber": "REF123456",
          "RegulatedIndicator": "Y",
          "RegulatedIndicatorDesc": "Regulated",
          "ReversalFlag": "N",
          "ReversalFlagDesc": "No",
          "SubmittedInterchangeID": "IC001",
          "SubmittedInterchangeDesc": "Standard Interchange",
          "TerminalID": "TERM001",
          "TotalAuthorizedAmount": "436.60",
          "TransactionAmount": "436.60",
          "TransactionCode": "1234",
          "TransactionDate": "09082026",
          "TransactionDecode": "Sale Transaction",
          "TransactionFeeAmount": "6.50",
          "TransactionFeeDebitCreditIndicator": "D",
          "TransactionFeeDebitCreditIndicatorDesc": "Debit",
          "VisaFeeProgramIndicator": "VFP01",
          "VisaIntegrityFee": "0.50"
        }
      ],
      "Adjustments": [
        {
          "ACHFlag": "Y",
          "ACHFlagDesc": "Yes",
          "AdjustmentAmount": "25.00",
          "ChargebackCaseNumber": "CB12345",
          "DebitCreditIndicator": "C",
          "DebitCreditIndicatorDesc": "Credit",
          "MerchantAccountNumber": "3948123412341234",
          "PostingDate": "09082026",
          "RecordIdentifier": "AD",
          "ReferenceNumber": "ADJREF123",
          "ReversalFlag": "N",
          "ReversalFlagDesc": "No",
          "TransactionCode": "1234",
          "TransactionDate": "09082026"
        }
      ]
    }
  ]
}
```

### Root response object

| Field | Type | Description |
|---|---|---|
| `Pagination` | object | Pagination metadata. |
| `BatchHeaders` | array | Batch header objects. |

### Pagination object

| Field | Type | Description |
|---|---|---|
| `PageNumber` | integer | Current page number. |
| `PageSize` | integer | Number of records requested per page. |
| `TotalRecords` | integer | Total available transaction records. |
| `TotalPages` | integer | Total pages based on the page size. |
| `HasPreviousPage` | boolean | `true` when a previous page exists. |
| `HasNextPage` | boolean | `true` when a next page exists. |

### Batch header object

| Field | Type | Description |
|---|---|---|
| `ACHPostDate` | string | ACH posting date in `MMDDYYYY` format. |
| `Adjustments` | array | Adjustment objects for the batch. |
| `BatchDate` | string | Batch processing date in `MMDDYYYY` format. |
| `BatchId` | string | Unique batch identifier. |
| `DebitCreditIndicator` | string | `D` for debit or `C` for credit. |
| `DebitCreditIndicatorDesc` | string | Debit or credit indicator description. |
| `MerchantId` | string | 16-digit Merchant ID. |
| `MerchantReferenceNumber` | string | Merchant-assigned batch reference number. |
| `NetDeposit` | string | Net deposit amount for the batch in decimal-dollar values. |
| `RecordType` | string | Batch header record type. Always `BH`. |
| `RejectReasonCode` | string | Batch rejection reason code, when applicable. |
| `Transactions` | array | Transaction objects for the batch. |

### Transaction object

| Field | Type | Description |
|---|---|---|
| `ACHFlag` | string | Indicates whether the merchant receives daily ACH funding. |
| `AmexSENumber` | string | American Express Service Establishment number. |
| `AuthAmount` | string | Authorization amount in decimal-dollar values. |
| `AuthCurrencyCode` | string | Three-character ISO authorization currency code. |
| `AuthCurrencyDecode` | string | Currency description, such as US Dollar or Canadian Dollar. |
| `AuthNumber` | string | Issuer authorization code. |
| `BanknetReferenceNumber` | string | Network-specific reference number. |
| `CardBrandFeeCode` | string | Card brand fee type code. |
| `CardBrandFeeDesc` | string | Card brand fee type description. |
| `CardholderAccountNumber` | string | Masked cardholder account number. |
| `CardTypeID` | string | Card type identifier. |
| `CardTypeDesc` | string | Card type description. |
| `CEDPIndicator` | string | Consumer Electronic Debit Program indicator. |
| `CommercialCardIndicator` | string | Commercial card indicator. |
| `CommercialCardIndicatorDesc` | string | Commercial card type description. |
| `CrossBorderFeeAmount` | string | Mastercard Cross Border, Visa ISA ASSM, or Amex Inbound fee amount. |
| `DBAName` | string | Merchant Doing Business As name. |
| `DebitCreditIndicator` | string | `D` for debit or `C` for credit. |
| `DebitCreditIndicatorDesc` | string | Debit or credit indicator description. |
| `ExtensionRecordIndicator` | string | Extension record indicator. |
| `ForeignExchangeFlag` | string | Indicates a foreign exchange transaction. |
| `GatewayReferenceId` | string | Core or PCON reference ID. Blank if the transaction was not processed through the Nuvei Gateway. |
| `IASFFeeAmount` | string | International Acquirer Service fee amount. |
| `IASFFeeDebitCreditIndicator` | string | Debit or credit indicator for the IASF fee. |
| `IASFFeeType` | string | International Acquirer Service fee type. |
| `InterchangeFeeAmount` | string | Interchange fee amount. |
| `InterchangeFeePercentRate` | string | Interchange fee percentage rate. |
| `InterchangePerItemRate` | string | Interchange flat per-item rate. |
| `MCCashBackFee` | string | Mastercard cash back fee amount. |
| `MCCashBackFeeSign` | string | Sign for the Mastercard cash back fee. |
| `MerchantAccountNumber` | string | 16-digit merchant account number. |
| `MerchantReferenceNumber` | string | Merchant-assigned reference number. |
| `NetDepositAmount` | string | Net deposit amount after fees, in decimal-dollar value. |
| `NetDepositAdjustmentAmount` | string | Net deposit adjustment amount. |
| `NetDepositAdjustmentDebitCredit` | string | Debit or credit indicator for the net deposit adjustment. |
| `NetworkDebitIdentifier` | string | Network that processed the debit transaction. |
| `OriginalTransactionAmount` | string | Original transaction amount before adjustments. |
| `ProductID` | string | Product identifier from `TblARDEFProduct` or `TBLGCMSProduct`. |
| `ProductIDDesc` | string | Product description. |
| `PurchaseID` | string | Merchant-assigned purchase identifier. |
| `RecordType` | string | Detail record type. Always `D`. |
| `ReferenceNumber` | string | Unique transaction reference number. |
| `RegulatedIndicator` | string | Regulated debit indicator. Values vary by card brand. |
| `RegulatedIndicatorDesc` | string | Regulated debit status description. |
| `ReversalFlag` | string | Indicates whether this is a reversal. |
| `ReversalFlagDesc` | string | Reversal flag description. |
| `SubmittedInterchangeID` | string | Interchange qualification level ID. |
| `SubmittedInterchangeDesc` | string | Interchange level description. |
| `TerminalID` | string | Point-of-sale terminal identifier. |
| `TotalAuthorizedAmount` | string | Total authorized amount including adjustments. |
| `TransactionAmount` | string | Transaction amount in cents as an 11-digit value. |
| `TransactionCode` | string | Four-digit transaction type code. |
| `TransactionDate` | string | Original transaction date in `MMDDYYYY` format. |
| `TransactionDecode` | string | Human-readable transaction code description. |
| `TransactionFeeAmount` | string | Transaction fee with a leading sign and two decimal places. |
| `TransactionFeeDebitCreditIndicator` | string | Debit or credit indicator for the transaction fee. |
| `TransactionFeeDebitCreditIndicatorDesc` | string | Transaction fee debit or credit description. |
| `VisaFeeProgramIndicator` | string | Visa fee program indicator. |
| `VisaIntegrityFee` | string | Visa integrity fee amount. |

### Adjustment object

| Field | Type | Description |
|---|---|---|
| `ACHFlag` | string | Indicates whether the merchant receives daily ACH funding. |
| `ACHFlagDesc` | string | ACH flag description. |
| `AdjustmentAmount` | string | Adjustment amount in decimal-dollar format as an 11-digit value. |
| `ChargebackCaseNumber` | string | Chargeback case number linking to the chargeback file. |
| `DebitCreditIndicator` | string | `D` for debit or `C` for credit. |
| `DebitCreditIndicatorDesc` | string | Debit or credit indicator description. |
| `MerchantAccountNumber` | string | 16-digit merchant account number. |
| `PostingDate` | string | Posting date in `MMDDYYYY` format. |
| `RecordIdentifier` | string | Adjustment record type. Always `AD`. |
| `ReferenceNumber` | string | 11-digit reference number linking to the original transaction. |
| `ReversalFlag` | string | Indicates whether this reverses a previous adjustment. |
| `ReversalFlagDesc` | string | Reversal flag description. |
| `TransactionCode` | string | Four-digit adjustment transaction type code. |
| `TransactionDate` | string | Original transaction date in `MMDDYYYY` format. |

## Error responses

### HTTP status codes

| Status | Description | When it occurs |
|---|---|---|
| `200 OK` | Successful request | The request was processed successfully. |
| `400 Bad Request` | Invalid request or validation error | The request body fails validation, the `Merchant-Auth` header is malformed, or `DeveloperId` is missing for a PCON merchant. |
| `401 Unauthorized` | Authentication failed | The `Merchant-Auth` header is invalid or missing. |
| `403 Forbidden` | Access unavailable | Access is not enabled for the user. |
| `500 Internal Server Error` | Server error | An unexpected processing error occurred. |

### Error body examples

#### 400 Bad Request

```json
{
  "message": "Validation failed",
  "errors": [
    {
      "field": "DateFrom",
      "errors": [
        "DateFrom must be in MMddyyyy format"
      ]
    },
    {
      "field": "MerchantAccountNumber",
      "errors": [
        "MerchantAccountNumber is required"
      ]
    }
  ]
}
```

#### 401 Unauthorized

```json
{
  "message": "Authentication failed"
}
```

#### 403 Forbidden

```json
{
  "message": "Feature not available yet"
}
```

#### 500 Internal Server Error

```json
{
  "message": "An error occurred while processing your request"
}
```

### Common validation errors

| Field | Error | Resolution |
|---|---|---|
| `MerchantAccountNumber` | `MerchantAccountNumber is required` | Provide a valid merchant account number. |
| `MerchantAccountNumber` | `MerchantAccountNumber must be 16 numbers` | Provide exactly 16 numeric digits. |
| `MerchantAccountNumber` | `MerchantAccountNumber must contain only numbers` | Remove nonnumeric characters. |
| `DateFrom` | `DateFrom must be in MMddyyyy format` | Use `MMDDYYYY`, for example `09082026`. |
| `DateFrom` | `DateFrom cannot be more than 90 days in the past. The earliest allowed date is {date}.` | Provide a date within the last 90 days. |
| `DateFrom` | `DateFrom cannot be in the future.` | Provide today's date or an earlier date. |
| `DateFrom` | `DateFrom must be less than or equal to DateTo.` | Ensure `DateFrom` is no later than `DateTo`. |
| `DateTo` | `DateTo must be in MMddyyyy format` | Use `MMDDYYYY`, for example `09142026`. |
| `DateTo` | `DateTo cannot be more than 90 days in the past. The earliest allowed date is {date}.` | Provide a date within the last 90 days. |
| `DateTo` | `DateTo cannot be in the future.` | Provide today's date or an earlier date. |
| `DateFrom`, `DateTo` | `The date range between DateFrom and DateTo cannot exceed 31 days.` | Request a date range of 31 days or less. |
| `DeveloperId` | `DeveloperId is required for PCON merchants` | Include `DeveloperId` in `Merchant-Auth` when `MerchantType` is `PCON`. |

## Code examples

The source document identifies both endpoints as `POST`; the examples below use `POST` consistently.

### cURL: deposits

```bash
curl --request POST \
  "https://www.sagepayments.net/web_services/Reconciliation/v1/deposits" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header 'Merchant-Auth: {"UserId":"GATEWAY_ID","UserKey":"GATEWAY_KEY","MerchantType":"CoreGateway"}' \
  --data '{
    "MerchantAccountNumber": "3948123412341234",
    "DateFrom": "09012026",
    "DateTo": "09142026"
  }'
```

### cURL: transactions

```bash
curl --request POST \
  "https://www.sagepayments.net/web_services/Reconciliation/v1/transactions" \
  --header "Content-Type: application/json" \
  --header "Accept: application/json" \
  --header 'Merchant-Auth: {"UserId":"GATEWAY_ID","UserKey":"GATEWAY_KEY","MerchantType":"CoreGateway"}' \
  --data '{
    "MerchantAccountNumber": "3948123412341234",
    "DateFrom": "09012026",
    "DateTo": "09142026",
    "PageNumber": 1,
    "PageSize": 3000
  }'
```
