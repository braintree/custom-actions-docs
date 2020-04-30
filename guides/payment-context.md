[&lt; Home](../README.md)

# Payment Contexts

A payment context is an object that can be created to store data about a payment method outside of the typical Braintree transaction flow. This is useful in cases where a payment may not be Authorized until asynchronous client-side flows are completed. This guide will walk through the steps of creating a Payment Context and tokenizing its data so that it can be used in Custom Actions event handlers such as Authorization and Capture.

<details>
<summary<strong>Contents</strong></summary>

- [Payment Context](#payment-contexts)
  - [Requirements](#requirements)
  - [Create a payment context](#create-a-payment-context)
    - [Mutation](#mutation)
    - [Variables](#variables)
  - [Create payment context event handler](#create-payment-context-event-handler)
    - [Handler](#handler)
  - [Accessing the payment context](#accessing-the-payment-context)
    - [Mutation](#mutation-1)
    - [Variables](#variables-1)

## Requirements

- A [Braintree sandbox](https://www.braintreepayments.com/sandbox) account
- Access to Custom Actions – (email custom-actions-requests@braintreepayments.com for access)
- A custom actions project - Reference the [Accept a new payment method](./accept-a-new-payment-method.md) guide.

## Create a payment context

To create a payment context, use the mutation below. This will trigger the `CreatePaymentContext` event handler associated with your Custom Actions project.

### Mutation

```graphql
mutation CreateCustomActionsPaymentContext(
  $input: CreateCustomActionsPaymentContextInput!
) {
  createCustomActionsPaymentContext(input: $input) {
    id
    createdAt
    customFields {
      name
      value
    }
  }
}
```

### Variables

```json
{
  "input": {
    "actionName": "your-custom-actions-project-name",
    "customFields": [
      {
			  "name": "custom-field-name",
			  "value": "custom-field-value"
		  }
    ]
  }
}
```

## Create payment context event handler

*Note*: A payment context can have up to a maximum of 5 custom fields.

Your Custom Actions `CreatePaymentContext` handler will be triggered when creating a payment context. When generating a new Custom Actions project a default `CreatePaymentContext` handler will be generated in your template. This handler will provide you with all the data you sent in the `CreatePaymentContext` mutation:

### Handler

```typescript
export const CreatePaymentContext = async (
  paymentContext: CreateBraintreePaymentContextInput
): Promise<BraintreeEventHandlerResponse> => {
  // possibly call out to a third-party here
  // to help generate your payment context

  return {
    paymentContextOrError: {
      customFields: [
        // Note: there is a maximum of 5 custom fields per payment context
        { name: "someCustomField", value: "someCustomValue" }
      ]
    }
  };
};
```

## Accessing the payment context

After creating a payment context, it can be retrieved using the node query, and the payment context ID provided after creation:

### Mutation

```graphql
query Node($id: ID!) {
  node(id: $id) {
    ... on CustomActionsPaymentContext {
      id
      createdAt
      updatedAt
      customFields {
        name
        value
      }
    }
  }
}
```

### Variables

```json
{
  "input": {
    "id": "your-payment-context-id",
  }
}
```
