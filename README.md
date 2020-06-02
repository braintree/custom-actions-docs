# The Custom Actions docs have moved to https://braintree.gitbook.io/custom-actions/

---

# <img src="https://user-images.githubusercontent.com/1217116/73565740-d2d7e900-4427-11ea-8d5b-b5692c425de0.png" alt="drawing" width="42" /> Custom Actions

<sup>**Last edit:** 05/01/2020</sup>

> Availability: We're building Custom Actions with the same proven expertise and rock-solid foundations you expect from Braintree.
>
> Email custom-actions-requests@braintreepayments.com to learn more.

Custom Actions is a serverless platform that opens the Braintree ecosystem to you and your developers by providing the capability to extend our built-in offerings. With Custom Actions, you can add new payment methods we don't currently support, send transaction data to your accounting systems, run fraud tools from the vendor of your choice, or develop a custom workflow unique to your business.

<details>
<summary><strong>Contents</strong></summary>

- [<img src="https://user-images.githubusercontent.com/1217116/73565740-d2d7e900-4427-11ea-8d5b-b5692c425de0.png" alt="drawing" width="42" /> Custom Actions](#img-src%22httpsuser-imagesgithubusercontentcom121711673565740-d2d7e900-4427-11ea-8d5b-b5692c425de0png%22-alt%22drawing%22-width%2242%22--custom-actions)
  - [What is it?](#what-is-it)
  - [Benefits](#benefits)
    - [Single integration](#single-integration)
    - [Unified reporting](#unified-reporting)
  - [Use cases](#use-cases)
    - [Payment methods](#payment-methods)
    - [Fraud tools](#fraud-tools)
    - [Data import & export](#data-import--export)
  - [Getting Started](#getting-started)
    - [Requirements](#requirements)
  - [Guides](#guides)
    </details>

## What is it?

We've defined **triggers** within Braintree that can execute code you've deployed into our platform. This code, called a **Custom Action** behaves as an **Event Handler** and is responsible for interacting with partners and vendors connected with your business. Often these handlers are simply mapping data from a third-party source to Braintree specific data.

Custom Actions focuses on developer experience as a first-class value. We provide tools to allow you to iterate quickly and deploy seamlessly. At its core, Custom Actions uses GraphQL and TypeScript to ensure type safety for your integration. These technologies also allow you to develop your integration locally within your own environment and leverage familiar tooling.

## Benefits

### Single integration

Payment methods run through Custom Actions are actionable within Braintree, just like any other. For example, you can initiate refunds or settle transactions the same as you would for a card payment.

### Unified reporting

By consolidating all of your transaction data within Braintree, you can view all of your commerce data in one place.
Globally available
Custom Actions are deployed globally. This means you can tailor your commerce to specific regions and enhance the experience for your users by minimizing latency.

## Use cases

![ca-diagram](https://user-images.githubusercontent.com/1217116/73570150-695cd800-4431-11ea-8bc0-6b3ad83c9d79.png)

### Payment methods

With Custom Actions, you can accept a payment method Braintree doesn’t support or initiate complex workflows with third-parties such as Twilio to take payment information over the phone.

### Fraud tools

You can use Custom Actions to run transactions through a fraud service.

### Data import & export

_Coming soon_

## Getting Started

### Requirements

- A Braintree sandbox account – ([signup here](https://www.braintreepayments.com/sandbox))
- Access to Custom Actions – (email custom-actions-requests@braintreepayments.com for access)
- Node.js v10.16 or later ([download](https://nodejs.org/en/download/))
  - [nvm](https://github.com/nvm-sh/nvm) is recommended to manage Node.js versions.
- Familiarity with [git](https://git-scm.com/) and command-line tools

## Guides

- [Accept a new payment method](./guides/accept-a-new-payment-method.md)
- [Payment Context](./concepts/payment-context.md)
