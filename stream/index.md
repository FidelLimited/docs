<div class="row">
  <div class="column">
    <a href="/stream/getting-started" class="content" data-path="/getting-started">
      <img src="https://docs.fidel.uk/assets/images/svgs/get-started-icon.svg" />
      <h2 data-no-link>Get Started</h2>
      <h3>Start building card-linked applications with our quickstart guide</h3>
    </a>
  </div>
  <div class="column">
    <a href="https://dashboard.fidel.uk/playground" class="content">
      <img src="https://docs.fidel.uk/assets/images/svgs/playground-icon.svg" />
      <h2 data-no-link>API Playground</h2>
      <h3>Test API requests in real-time and see how we format the returned data.</h3>
    </a>
  </div>
</div>
<div class="row">
  <div class="column">
    <a href="/stream/tutorials/card-linking" class="content">
      <img src="https://docs.fidel.uk/assets/images/svgs/tutorials-icon.svg" />
      <h2 data-no-link>Tutorials</h2>
      <h3>Learn how to build a card-linking feature into your applications.</h3>
    </a>
  </div>
  <div class="column">
    <a href="/stream/sdks/web/v3" data-path="/stream/sdks/web/v3" class="content">
      <img src="https://docs.fidel.uk/assets/images/svgs/sdks-icon.svg" />
      <h2 data-no-link>SDKs</h2>
      <h3>Take the easy and secure way to add card enrollment capabilities into your application.</h3>
    </a>
  </div>
</div>

## Introduction

Fidel API provides a developer-friendly, secure and reliable API to link payment cards to mobile and web applications. Through a single API, developers can securely access real-time data from the card networks and build applications on top of a powerful financial infrastructure platform.

Once a card is enrolled and verified, every payment event (i.e. authorization, clearing, refund) from that card will be sent to your server in real time. The transaction data you receive is of high quality, accuracy and consistency. This allows you to build a rich experience for cardholders at the moment the transaction is made, while they are still at the terminal or checkout page.

## Transaction Life Cycle

To understand how the Transaction Stream API works and the data it provides to you, it’s important to understand the authorization processes and fund movements that happen when a cardholder makes a purchase.

### Authorization, clearing

When a cardholder uses a credit or debit card to make a purchase, the funds are not immediately transferred to the merchant’s account. There are two important events that need to happen for the funds to be transferred: authorization and clearing. Here’s how these events occur:

1. When the cardholder initiates the transaction, their bank (issuing bank, issuer) needs to authorize it. For this, the **authorization request** must travel from the merchant through the merchant’s bank (acquirer) and through the card network to the issuing bank.  
   ![](https://docs.fidel.uk/assets/images/trx-life-cycle-1.png)
2. If the cardholder has the necessary funds, the issuing bank sends back on the same path the **authorization response** containing the authorization code (auth code), which means that the cardholder can make the purchase.  
   At this point, the payment amount is still on the cardholder’s account. However, the merchant can safely provide the purchased goods or services, as the transaction was authorized. Usually, the merchant places an authorization hold on the cardholder’s account for the authorized amount of the sale.  
   ![](https://docs.fidel.uk/assets/images/trx-life-cycle-2.png)
3. The next step is the **clearing request**, which initiates the administrative process of the payment.  
   Typically, clearing occurs at the end of the day, when the acquirer bank collects all the transaction information (amounts, auth codes, etc.) from all payment endpoints of the merchant. Then, on its own processing schedule, the acquirer starts processing the payments with the respective issuing banks.  
   ![](https://docs.fidel.uk/assets/images/trx-life-cycle-3.png)
4. The issuing bank sends back the **clearing response**, and the funds are moved to the merchant’s account.  
   ![](https://docs.fidel.uk/assets/images/trx-life-cycle-4.png)

Note that at Fidel API both cleared and settled transactions are referred to as cleared transactions.

### Where does Fidel API enter the picture?

As we saw, all the transaction events go through the card networks. This is where Fidel API comes into play.

When a cardholder links a card to a Fidel API program, Fidel API verifies the card with the associated card network and creates a token to represent that card, thereby not storing sensitive information about the cardholder or card itself. Using this token-based identification, after you verify that the user has access to the card, the card network starts sending the transactions made on that card to Fidel API.

You can then retrieve the collected data using the Transaction Stream API. You can also register webhooks to be notified about the card’s transaction events (new authorization, clearing, etc.).

![](https://docs.fidel.uk/assets/images/trx-life-cycle-5.png)

## Collected Data

The Transaction Stream API collects the following data for the supported transactions made with linked cards:

| **Field name**                                                                  | **Description**                                                                                                                                                                                                                                                                                                                                  | **Type**                                                            |
| ------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ | ------------------------------------------------------------------- |
| `accountId`                                                                     | The unique identifier of the user account at Fidel API.                                                                                                                                                                                                                                                                                          | string                                                              |
| `amount`                                                                        | The amount of the transaction in the currency it was charged in. In the case of international transactions, the transaction has both `amount` and `billingAmount` fields filled in and their value may differ due to the conversion.                                                                                                             | number                                                              |
| `auth`                                                                          | Indicates whether the transaction was authorized by the issuing bank.                                                                                                                                                                                                                                                                            | boolean                                                             |
| `authCode`                                                                      | Unique code that authorizes a transaction, sent by the issuer/the PCN on the issuer's behalf when approving a transaction. It mirrors the `identifiers.visaAuthCode` property. You can use the authorization code in combination with the card number to match an authorization event and a clearing event when the transaction ID is different. | string (or null)                                                    |
| `billingAmount`                                                                 | The amount in the currency of the country of issuance, converted from the original purchase currency. In the case of international transactions, the transaction has both `amount` and `billingAmount` fields filled in and their value may differ due to the conversion.                                                                        | number (or null)                                                    |
| `billingCurrency`                                                               | The currency of the country of issuance.                                                                                                                                                                                                                                                                                                         | string in ISO 4217 format (or null)                                 |
| `brand.name`                                                                    | The name of the merchant where the purchase was made.                                                                                                                                                                                                                                                                                            | string                                                              |
| `card.firstNumbers`                                                             | The first numbers of the card, used for helping users identify their cards.                                                                                                                                                                                                                                                                      | string                                                              |
| `card.id`                                                                       | The tokenized card identifier used by Fidel API.                                                                                                                                                                                                                                                                                                 | string                                                              |
| `card.lastNumbers`                                                              | The last numbers of the card, used for helping users identify their cards.                                                                                                                                                                                                                                                                       | string                                                              |
| `card.scheme`                                                                   | The payment network (Visa).                                                                                                                                                                                                                                                                                                                      | string                                                              |
| `cardPresent`                                                                   | Not in use                                                                                                                                                                                                                                                                                                                                       | null                                                                |
| `cleared`                                                                       | Indicates whether the transaction was cleared.                                                                                                                                                                                                                                                                                                   | boolean                                                             |
| `created`                                                                       | The timestamp in UTC format when the transaction object was created in the Fidel API database.                                                                                                                                                                                                                                                   | string, timestamp in UTC format                                     |
| `currency`                                                                      | The original currency of the purchase.                                                                                                                                                                                                                                                                                                           | string in [ISO 4217](https://en.wikipedia.org/wiki/ISO_4217) format |
| `datetime`                                                                      | The timestamp (in the local timezone of the merchant/acquirer) of the transaction.                                                                                                                                                                                                                                                               | string, timestamp in UTC format                                     |
| `id`                                                                            | The unique identifier of the transaction.                                                                                                                                                                                                                                                                                                        | string                                                              |
| `identifiers.amexApprovalCode`                                                  | Not in use in Transaction Stream programs.                                                                                                                                                                                                                                                                                                       | null                                                                |
| `identifiers.mastercardAuthCode`                                                | Not in use in Transaction Stream programs.                                                                                                                                                                                                                                                                                                       | null                                                                |
| `identifiers.mastercardRefNumber`                                               | Not in use in Transaction Stream programs.                                                                                                                                                                                                                                                                                                       | null                                                                |
| `identifiers.mastercardTransactionSequenceNumber`                               | Not in use in Transaction Stream programs.                                                                                                                                                                                                                                                                                                       | null                                                                |
| `identifiers.visaAuthCode`                                                      | Unique code that is sent along with the authorization of the transaction by Visa. It mirrors the `authCode` property.                                                                                                                                                                                                                            | string (or null)                                                    |
| `location`                                                                      | Details of the merchant where the purchase was made:                                                                                                                                                                                                                                                                                             |
| `location.city`, `location.countryCode`, `location.postcode`, `location.state`. | string (city, postcode, and state can be null)                                                                                                                                                                                                                                                                                                   |
| `merchantCategoryCode`                                                          | The merchant category code (MCC) provided by the card network. It is used to classify businesses by the types of goods provided or services rendered.                                                                                                                                                                                            | string                                                              |
| `programId`                                                                     | The identifier of the Fidel API program that this transaction is linked to.                                                                                                                                                                                                                                                                      | string                                                              |
| `programType`                                                                   | The type of the Fidel API program that this transaction is linked to (`transaction-stream`).                                                                                                                                                                                                                                                     | string                                                              |
| `updated`                                                                       | The date and time when this transaction was last time updated (authorized/cleared/reimbursed).                                                                                                                                                                                                                                                   | string, timestamp in UTC format                                     |
| `wallet`                                                                        | Not in use.                                                                                                                                                                                                                                                                                                                                      | null                                                                |

You can see the JSON format of an example transaction object in the [Transactions](/stream/transactions) page.

## Learn More

Start your Fidel API journey by creating an account. Read the [Getting Started](/stream/getting-started) page to learn how.

You can see an example implementation for integrating the Fidel APIs and Web SDK in our [sample application on GitHub](https://github.com/FidelLimited/fidel-api-sample-app).

Check out the [API Reference](https://transaction-stream.fidel.uk/reference/introduction-1) to see all available requests, code examples and response payloads.
