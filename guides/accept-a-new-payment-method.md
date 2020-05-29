[&lt; Home](../README.md)

# Accept a new payment method

This guide will walk through the steps of creating a Custom Actions payment method, wiring up a client, and deploying your project for use in your sandbox account.

By default, a project provides **hooks** for the [authorize](#authorization), [capture](#capture), [void](#void), and [refund](#refund) phases of a transaction as well as a **hook** for creating a [Payment Context](./create-a-payment-context.md).

You can learn more about the lifecycle of a Transaction [here](https://developers.braintreepayments.com/guides/transactions/node).

> **Note:** Custom Actions provides pre-defined project templates. Not every integration will fall neatly within a single template, but for those, you can create a custom integration or reach out to custom-actions-requests@braintreepayments.com

<details>
<summary><strong>Contents</strong></summary>

- [Accept a new payment method](#accept-a-new-payment-method)
  - [Requirements](#requirements)
  - [Request Custom Actions access](#request-custom-actions-access)
  - [Create a project](#create-a-project)
    - [Setting up your development environment](#setting-up-your-development-environment)
      - [Install dependencies](#install-dependencies)
      - [Start development mode](#start-development-mode)
    - [Preparing secrets](#preparing-secrets)
  - [Write your event handlers](#write-your-event-handlers)
    - [Authorization handler](#authorization-handler)
      - [Invoking the Authorization handler](#invoking-the-authorization-handler)
    - [Capture handler](#capture-handler)
      - [Invoking the Capture handler](#invoking-the-capture-handler)
    - [Reversal handlers](#reversal-handlers)
      - [Void Handler](#void-handler)
      - [Refund Handler](#refund-handler)
    - [Create Payment Context Handler](#create-payment-context-handler)
      - [Error handling](#error-handling)
    - [Resolve Payment Context Handler](#resolve-payment-context-handler)
  - [Build your client-side integration](#build-your-client-side-integration)
    - [Tokenization](#tokenization)
  - [Deploy your custom action](#deploy-your-custom-action)
  - [Monitor your Custom Action](#monitor-your-custom-action)
    - [Viewing logs](#viewing-logs)
  - [Going to production](#going-to-production)
      </details>

## Requirements

To build a new payment method integration, you will need:

- A [Braintree sandbox](https://www.braintreepayments.com/sandbox) account
- Access to Custom Actions – (email custom-actions-requests@braintreepayments.com for access)
- Node.js v10.16 or later ([download](https://nodejs.org/en/download/))
  - [nvm](https://github.com/nvm-sh/nvm) is recommended to manage Node.js versions.
- The [Braintree CLI](../cli/index.md)

It would also be helpful to familiarize yourself with our [GraphQL API](https://graphql.braintreepayments.com).

## Request Custom Actions access

Custom Actions needs to be enabled for each merchant account. Send an email to custom-actions-requests@braintreepayments.com for access. Here is a handy template that you can use:

> Hello Custom Actions Team!
>
> I would like to enable Custom Actions for my merchant.
>
> My sandbox merchant ID is: YOUR_SANDBOX_MERCHANT_ID

## Create a project

Ensure you have installed the [CLI](../cli/index.md) and run `btx actions:create`.

This will prompt your for:

- A **project name:** – This is a unique name for your Custom Action and usually references the payment method you are creating.
- Your **merchant id:** – This is your sandbox merchant ID ([Learn how to find your account credentials](https://articles.braintreepayments.com/control-panel/important-gateway-credentials#merchant-id)).

```sh
~% btx actions:create
> Name your project: foopay
> Enter your Braintree merchant id: 92ytw34t4t3qt6gk
  ✔ Creating project
  ✔ Preparing template
Created foopay (vcGxOOF2) in /user/you

To get started, run: `cd /user/you/foopay && bt deploy`
```

You should now see a new directory called `foopay` (or whatever you named your project). `cd` into that directory and you will see your event handlers have been scaffolded out for you in the `src` directory.

```
foopay/src
└── handlers
    ├── AuthorizeTransaction.test.ts
    ├── AuthorizeTransaction.ts
    ├── CaptureTransaction.test.ts
    ├── CaptureTransaction.ts
    ├── CreatePaymentContext.test.ts
    ├── CreatePaymentContext.ts
    ├── ResolvePaymentContext.test.ts
    ├── ResolvePaymentContext.ts
    ├── RefundTransaction.test.ts
    ├── RefundTransaction.ts
    ├── VoidTransaction.test.ts
    └── VoidTransaction.ts
```

### Setting up your development environment

Custom Actions projects are written in [TypeScript](https://www.typescriptlang.org/). Type definitions and unit tests are provided out of the box – along with a "watch" mode that will rebuild your project as you update your code.

#### Install dependencies

```sh
$ npm i
```

#### Start development mode

```sh
$ npm run test:watch
```

Now that your environment is running, you can begin wiring up your integration.

### Preparing secrets

Your integration will likely require access to sensitive values such as API keys. You can securely store secrets for use in your Custom Action using the CLI.

```sh
$ btx actions:secrets:add MY_KEY MY_VALUE
```

This will create a secret named `MY_KEY` and with the value `MY_VALUE`. This secret can be accessed in your code via an environment variable – `process.env.MY_KEY`.

See the [CLI reference](https://github.braintreeps.com/braintree/cli) (GHE link) for more details.

## Write your event handlers

### Authorization handler

When you [authorize](https://graphql.braintreepayments.com/guides/creating_transactions#using-separate-authorization-and-capture) a payment method via the [Braintree API](https://graphql.braintreepayments.com) using a [tokenized Custom Actions payment method](#tokenization), your `authorizeTransaction` event handler will be called. This handler will provide you with a [Transaction](./reference.md) Object and expect you to return a [TransactionStatusEvent](./reference) with a new [status](https://github.com/braintree/braintree-types/blob/9197377866c59f5465b115099f69a8c3242b7f16/src/index.ts#L298).

> **Note:** When using [`chargePaymentMethod`](https://graphql.braintreepayments.com/reference/#Mutation--chargePaymentMethod) via the GraphQL API or passing [`submitForSettlement: true`](https://developers.braintreepayments.com/reference/request/transaction/sale/node#options.submit_for_settlement) on a `transaction.sale()` call via the SDK, your `authorizeTransactionHandler` **_*and*_** your [`captureTransactionHandler`](#capture) will be invoked serially.

```js
export const AuthorizeTransactionHandler = async (
  transaction: BraintreeTransaction
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    transactionStatusEvent: {
      id: transaction.id,
      status: BraintreeTransactionStatus.AUTHORIZED,
    },
  };
};
```

#### Invoking the Authorization handler

This handler will be invoked when you call [`transaction.sale()`](https://developers.braintreepayments.com/reference/request/transaction/sale) through the SDK or [`authorizePaymentMethod`](https://graphql.braintreepayments.com/reference/#Mutation--authorizePaymentMethod) through the GraphQL API.

**Example GraphQL API call:**

```graphql
mutation ExampleAuth($input: AuthorizePaymentMethodInput!) {
  authorizePaymentMethod(input: $input) {
    transaction {
      id
      status
    }
  }
}
```

**variables**

```json
{
  "input": {
    "paymentMethodId": "tokencap_123",
    "transaction": {
      "merchantAccountId": "id_of_custom_action_enabled_merchant_account",
      "amount": "11.23"
    }
  }
}
```

**response**

```json
{
  "data": {
    "chargePaymentMethod": {
      "transaction": {
        "id": "id_of_authorized_transaction",
        "status": "AUTHORIZED"
      }
    }
  }
}
```

At this point, it may be a good idea to verify your integration is working for this first handler. Feel free to skip to [building your client-side integration](#build-your-client-side-integration) and running your first [deployment](#deploy-your-custom-action) before writing your other handlers.

### Capture handler

Capturing a transaction represents the [settlement](https://developers.braintreepayments.com/guides/transactions/node#settlement) phase which is required to actually collect payment from a previous [authorization](#authorization).

When calling [`transaction.submitForSettlement()`](https://developers.braintreepayments.com/reference/request/transaction/submit-for-settlement) through the SDK or [`captureTransaction`](https://graphql.braintreepayments.com/reference/#Mutation--captureTransaction) through the GraphQL API (preferred\), your `captureTransaction` event handler will be called. Like the authorize handler, this function will be called with a [Transaction](./reference.md) Object and expect you to return a [TransactionStatusEvent](./reference) with a new [status](https://github.com/braintree/braintree-types/blob/9197377866c59f5465b115099f69a8c3242b7f16/src/index.ts#L304-L308).

> **Note:** When using [`chargePaymentMethod`](https://graphql.braintreepayments.com/reference/#Mutation--chargePaymentMethod) via the GraphQL API or passing [`submitForSettlement: true`](https://developers.braintreepayments.com/reference/request/transaction/sale/node#options.submit_for_settlement) on a `transaction.sale()` call via the SDK, your [`authorizeTransactionHandler`](#authorization) **_*and*_** your `captureTransactionHandler` will be invoked serially.

If your payment method supports inline capture and does not need to be batched, you an return a `SETTLED` or `SETTLEMENT_PENDING` event. If your payment method needs to be batched at set intervals, use the `SUBMITTED_FOR_SETTLEMENT` event with a `settlementTimestamp`.

```js
export const CaptureTransactionHandler = async (
  transaction: BraintreeTransaction
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    transactionStatusEvent: {
      id: transaction.id,
      settlementTimestamp: new Date().toISOString(),
      status: BraintreeTransactionStatus.SUBMITTED_FOR_SETTLEMENT,
    },
  };
};
```

#### Invoking the Capture handler

**Example GraphQL API call:**

```graphql
mutation ExampleCapture($input: CaptureTransactionInput!) {
  captureTransaction(input: $input) {
    transaction {
      id
      status
    }
  }
}
```

**variables**

```json
{
  "input": {
    "transactionId": "id_of_authorized_transaction"
  }
}
```

**response**

```json
{
  "data": {
    "transaction": {
      "id": "id_of_transaction",
      "status": "SUBMITTED_FOR_SETTLEMENT"
    }
  }
}
```

### Reversal handlers

A transaction can be reversed by either _voiding_ the transaction if it has not be settled or _refunding_ the transaction if it has been settled.

Braintree's API exposes a [`reverseTransaction`](https://graphql.braintreepayments.com/guides/reversing_transactions/) mutation to handle the logic automatically depending on the state of the transaction. Depending on the state, one of your handlers will be invoked, either [`VoidTransactionHandler`](#void-handler) or [`RefundTransactionHandler`](#refund-handler).

Alternatively, if you are using the SDK, calling [`transaction.void()`](https://developers.braintreepayments.com/reference/request/transaction/void) or [`transaction.refund()`](https://developers.braintreepayments.com/reference/request/transaction/refund) will invoke these handlers respectively.

**Example GraphQL API call:**

```graphql
mutation ExampleReverse($input: ReverseTransactionInput!) {
  reverseTransaction(input: $input) {
    reversal {
      ... on Transaction {
        id
        status
        statusHistory {
          status
          terminal
        }
      }
      ... on Refund {
        id
        amount {
          value
        }
        orderId
        status
        refundedTransaction {
          id
          amount {
            value
          }
          orderId
          status
        }
      }
    }
  }
}
```

**variables**

```json
{
  "input": {
    "transactionId": "id_of_successful_transaction"
  }
}
```

**response**

```json
"data": {
  "reverseTransaction": {
    "reversal": {
      "id": "TRANSACTION_ID",
      "status": "VOIDED",
      "statusHistory": [
        {
          "status": "VOIDED",
          "terminal": true,
        },
        {
          "status": "SUBMITTED_FOR_SETTLEMENT",
          "terminal": false,
        },
        {
          "status": "AUTHORIZED",
          "terminal": false,
        }
      ]
    }
  }
}
```

#### Void Handler

A transaction in the [`AUTHORIZED`](https://developers.braintreepayments.com/reference/general/statuses#authorized) or [`SUBMITTED_FOR_SETTLEMENT`](https://developers.braintreepayments.com/reference/general/statuses#submitted-for-settlement) state can be voided. This will invoke the `voidHandler` function for your Custom Action with a [Transaction](./reference.md) Object and expect you to return a [TransactionStatusEvent](./reference) with a new status of `VOIDED`.

```js
export const VoidTransactionHandler = async (
  transaction: BraintreeTransaction
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    transactionStatusEvent: {
      id: transaction.id,
      status: BraintreeTransactionStatus.VOIDED,
    },
  };
};
```

#### Refund Handler

A transaction in the [`SETTLED`](https://developers.braintreepayments.com/reference/general/statuses#settled) or [`SETTLING`](https://developers.braintreepayments.com/reference/general/statuses#settling) state can be refunded. This will invoke the `refundTransaction` function for your Custom Action with a [Refund](./reference.md) Object and expect you to return a [TransactionStatusEvent](./reference) with a new status of `SUBMITTED_FOR_SETTLEMENT`.

```js
export const RefundTransactionHandler = async (
  transaction: BraintreeRefund
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    transactionStatusEvent: {
      id: transaction.id,
      settlementTimestamp: new Date(),
      status: BraintreeTransactionStatus.SUBMITTED_FOR_SETTLEMENT,
    },
  };
};
```

### Create Payment Context Handler

There are cases where you may need to track buyer activity or capture data _before_ creating a transaction. In these cases, you can use a **Payment Context**. With a **Payment Context**, you can send data to a Custom Actions handler, make a call to an API, and then store the result within Braintree for use on your client or server.

```typescript
export const CreatePaymentContextHandler = async (
  paymentContext: CreateBraintreePaymentContextInput
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    paymentContextOrError: {
      // Return fields to be stored on the Payment Context
      customFields: [
        {
          name: "orderId",
          value: myAPI.createOrder().id,
        },
      ],
    },
  };
};
```

#### Error handling

You may encounter a scenario, such as a validation error, where you need to return a message to a client. In that case, you can return a structured `Error` type from your handler:

```typescript
export const CreatePaymentContextHandler = async (
  paymentContext: CreateBraintreePaymentContextInput
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  return {
    paymentContextOrError: {
      message: "A valid address must be provided",
    },
  };
};
```

### Resolve Payment Context Handler

You can look up a Payment Context by passing its id to a [node query](https://graphql.braintreepayments.com/guides/node_query/). In these cases, a Custom Actions handler called `ResolvePaymentContext` will be called. In this handler, you may return the Payment Context itself, or if you need to negotiate state with a downstream service, you can make a call and then return the new fields.

```typescript
export const ResolvePaymentContextHandler = async (
  paymentContext: BraintreePaymentContextInput
): Promise<BraintreeEventHandlerResponse> => {
  // Call out to third-party here

  // tslint:disable-next-line:no-console
  console.log(paymentContext);

  return {
    paymentContextOrError: {
      customFields: [...(paymentContext.customFields || [])],
    },
  };
};
```

## Build your client-side integration

### Tokenization

Custom Actions works with existing Braintree integrations. For most payment methods, data is [tokenized](https://developers.braintreepayments.com/guides/payment-method-nonces) for use as a [Payment Method Nonce](https://developers.braintreepayments.com/guides/payment-method-nonces). For first-class payment methods, Braintree SDKs handle this tokenization for you. In the case of Custom Actions, you will need to tokenize the values specific to your integration and exchange them for a nonce.

This nonce will be decoded and the values provided to you in your [Authorization handler](#authorization-handler) so that you can pass them along to a downstream payment method's API.

The [Braintree GraphQL API](https://graphql.braintreepayments.com) provides a [`tokenizeCustomActionsPaymentMethod`](https://graphql.braintreepayments.com/reference/#Mutation--tokenizeCustomActionsPaymentMethod) mutation which can be called from your client.

**Example GraphQL API call:**

```graphql
mutation TokenizeCustomActionsPaymentMethod(
  $input: TokenizeCustomActionsPaymentMethodInput!
) {
  tokenizeCustomActionsPaymentMethod(input: $input) {
    paymentMethod {
      id
      details {
        ... on CustomActionsPaymentMethodDetails {
          actionName
          fields {
            name
            displayValue
          }
        }
      }
    }
  }
}
```

**variables**

```json
{
  "input": {
    "customActionsPaymentMethod": {
      "actionName": "foopay",
      "fields": [
        {
          "name": "accountNumber",
          "value": "12345",
          "displayValue": "****5"
        },
        {
          "name": "accountName",
          "value": "Brian Tree",
          "displayValue": "Brian Tree"
        }
      ]
    }
  }
}
```

**response**

```json
{
  "data": {
    "tokenizeCustomActionsPaymentMethod": {
      "paymentMethod": {
        "id": "tokencap_123",
        "details": {
          "actionName": "foopay",
          "fields": [
            {
              "name": "accountNumber",
              "displayValue": "****5"
            },
            {
              "name": "accountName",
              "displayValue": "Brian Tree"
            }
          ]
        }
      }
    }
  },
  "extensions": {
    "requestId": "some-unique-string"
  }
}
```

## Deploy your custom action

Once you are ready to run a test transaction through your integration, you can deploy your handlers. This process will package up your code and deploy it into Braintree's cloud, making it available through our existing APIs.

Using the CLI, from within your project directory, run:

```sh
btx actions:deploy
```

This will kick off the process and let you know when your deployment is ready.

```sh
~/foopay% btx actions:deploy

Having issues? Reach out to #service-custom-actions

  ✔ Parsing braintree.yml
  ✔ Building project
  ✔ Creating deployment resource
  ✔ Prepare artifact
  ✔ Deploying project
  ✔ Waiting for deploy status
Project deployed!
```

## Monitor your Custom Action

### Viewing logs

This feature is currently a work-in-progress. In the meantime, the Custom Actions team can help you view logs and debug integrations as needed.

## Going to production

TODO
