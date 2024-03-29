# Reimbursement

Fidel Reimbursement is an add-on product to _Transaction_ tracking capability, that gives developers the ability to reimburse customers faster and more easily than ever before, by pushing cash directly onto linked cards.

## Availability

Reimbursement supports both debit and credit cards by Visa and Mastercard in the US or UK. Your account must be set up either in the US or the UK to have access to the Reimbursement product.

## Activation

If your account is set up in the US or UK and you are using an older API version (reimbursement requires version 2021-09-28 or later) or haven't agreed to the [Terms & Conditions](https://fidelapi.com/legal) in your region, you will be prompted to go through the Reimbursement product activation.

![Reimbursement Activation](https://docs.fidel.uk/assets/images/gifs/reimbursement-activation.gif "Reimbursement Activation")

## Credits

Before reimbursing cardholders you will need to buy Fidel Credits. To be able to make a reimbursement request you need to have enough credits in your balance to deduct the reimbursement amount. Credits currently support USD or GBP currency denominations only and must be purchased by bank transfer to FIDEL LIMITED’s beneficiary account using your unique reference code in the description. Find yours under the [Credits view in the dashboard](https://dashboard.fidel.uk/account/credits).

Fidel Credits are non refundable and non transferable. Take into consideration possible bank transfer delays that can take between 2-3 business days.

![Credits view in the dashboard](https://docs.fidel.uk/assets/images/credits-view.png "Credits view in the dashboard")

### Balance

Your credit balance is updated every time you purchase credits or spend by using Reimbursement. When sending a reimbursement request, the amount is immediately deducted from the balance. If the reimbursement goes to `failed` status, the amount is added again to the balance. Read more information on the [credits balance endpoint](https://reference.fidel.uk/reference/get-account-balance).

### Credits Balance Example

Balances are denominated in multiple currencies to support other countries in the future. Currently, we support either `USD` or `GBP` and in this example `balances.USD` value is `$1000`.

```sh
curl -X GET \
  https://api.fidel.uk/v1/accounts/{accountId}/credits/balance \
  -H 'Content-Type: application/json' \
  -H 'Fidel-Key: <KEY>'
```

```json
fileName:credits-balance.json
{
  "items": [
    {
      "accountId": "61741c3-3dc9-45f5-8e7c-db1dd649afab",
      "balances": {
        "AUD": 0,
        "CHF": 0,
      "JPY": 0,
        "EUR": 0,
        "GBP": 0,
        "CAD": 0,
        "USD": 1000,
        "NZD": 0
      },
      "lastTotalAmount": {
        "AUD": 0,
        "CHF": 0,
        "JPY": 0,
        "EUR": 0,
        "GBP": 0,
        "CAD": 0,
        "USD": 1000,
        "NZD": 0
      },
      "lowCreditsNotificationSent": {
        "AUD": false,
        "CHF": false,
        "JPY": false,
        "EUR": false,
        "GBP": false,
        "CAD": false,
        "USD": true,
        "NZD": false
      }
    }
  ],
  "execution": 135.265058,
  "resource": "/v1/accounts/61741c3-3dc9-45f5-8e7c-db1dd649afab/credits/balance",
  "status": 200
}
```

<h3 id="low-balance-notification">Low Balance Notification</h3>

When a currency balance drops to _25%_ of the balance amount you had on your last purchase, `lowCreditsNotificationSent` is set to true and the following actions are then triggered:

- notification email is sent to account users emails;

- dashboard notification is shown;

- `credits.balance.low` webhook is sent if you are listening on your end. Read below on how to set it up.

### Low Balance Webhook

The `credits.balance.low` webhook notifies on the _[Low Balance Notification](/select/reimbursement/#low-balance-notification)_ event that happens when the credits balance is running low. Use this webhook to automate credit purchases on your end without the risk of service disruption due to insufficient balance. See the example below with the webhook object triggered by the low USD balance:

```json
fileName:credits-balance.json
{
  "accountId": "61741c3-3dc9-45f5-8e7c-db1dd649afab",
  "balances": {
    "AUD": 0,
    "CHF": 0,
    "JPY": 0,
    "EUR": 0,
    "GBP": 0,
    "CAD": 0,
    "USD": 1000,
    "NZD": 0
  },
  "lastTotalAmount": {
    "AUD": 0,
    "CHF": 0,
    "JPY": 0,
    "EUR": 0,
    "GBP": 0,
    "CAD": 0,
    "USD": 1000,
    "NZD": 0
  },
  "lowCreditsNotificationSent": {
    "AUD": false,
    "CHF": false,
    "JPY": false,
    "EUR": false,
    "GBP": false,
    "CAD": false,
    "USD": true,
    "NZD": false
  }
}
```

### History

In the Credits dashboard view you have access to your credit purchases in _Purchased credits_ and credit spent in _Reimbursements_. Read more information on the [credits history endpoint](https://reference.fidel.uk/reference/get-credits-history).

![Credits history](https://docs.fidel.uk/assets/images/gifs/credits-history.gif "Credits history")

## Eligibility

The reimbursement request is done towards a cardholder transaction with the parameter `transactionId` and it needs to meet the eligibility criteria.

A transaction is said to be eligible when it meets the following criteria:

- Transaction `currency` is `USD` or `GBP`;

- Transaction `reimbursement` is `undefined` or `reimbursement.status` is `failed`;

- Transaction `scheme` is `visa` or `mastercard`;

- Transaction status is `cleared`;

- Transaction `time` date is newer than 90 days.

### Finding Eligible Transactions by Card

Find eligible card transactions with `cardId` for a specific `amount` and `currency`. Optionally pass `brandId` to find transactions in a specific brand.

Transactions are sorted automatically in descending order by the best available, following the eligibility criteria, giving more emphasis on the transaction time and then amount. Read more information on the [find eligible reimbursement transactions endpoint](https://reference.fidel.uk/reference/get-transactions-for-reimbursement-by-card).

![Reimbursement by Card](https://docs.fidel.uk/assets/images/gifs/reimburse-card.gif "Reimbursement by Card")

### Eligible Transactions Example

Example `amount` set to `5`, `currency` to `USD` and `brandId` to `Star`.

```sh
curl -X GET \
  GET https://api.fidel.uk/v1/cards/{cardId}/transactions/reimbursement?amount={amount}&currency={currency}&brandId={brandId} \
  -H 'Content-Type: application/json' \
  -H 'Fidel-Key: <KEY>'
```

```json
fileName:reimburse-eligible-transactions.json
{
  "count": 2,
  "items": [
    {
      // For the purpose of this example, only selected properties are shown
      "amount": 10,
      "currency": "USD",
      "card": {
        "scheme": "visa",
      },
      "id": "cf0a2949-8603-4609-8d29-03b9ae5a446b",
      "accountId": "d1bc1828-f904-48a3-8492-12272c81c117",
      "programId": "92a10f41-cbe3-423d-a7b3-61ca4ce5e6c6"
    },
    {
      // For the purpose of this example, only selected properties are shown
      "amount": 20,
      "currency": "USD",
      "card": {
        "scheme": "visa",
      },
      "id": "53ad6957-3a28-499a-a206-558b89ca0d45",
      "accountId": "d1bc1828-f904-48a3-8492-12272c81c117",
      "programId": "92a10f41-cbe3-423d-a7b3-61ca4ce5e6c6"
    }
  ],
  "resource": "/v1/cards/bc538b71-31c5-4699-840a-6d4a08693314/transactions/reimbursement",
  "status": 200,
  "execution": 26.392104
}
```

## Request

### Creating a request

After choosing the `transactionId`, the reimbursement `amount` must be equal to or lower than the transaction `amount` and the `currency` is determined by the transaction. Visa cards have a maximum reimbursement amount limit of _USD $250_ or _GBP £250_, depending on the currency. We currently support `USD` or `GBP` currency transactions. Optionally customise the `description` text that will show in the cardholder bank statement. Read more information on the [create reimbursement endpoint](https://reference.fidel.uk/reference/create-reimbursement).

![Creating a reimbursement](https://docs.fidel.uk/assets/images/gifs/reimbursing.gif "Creating a reimbursement")

### Reimbursement Example

Example `amount` set to `2.55` and custom `description` to `Earned Stars`.

```sh
curl -X POST \
  https://api.fidel.uk/v1/transactions/{transactionId}/reimbursement \
  -H 'Content-Type: application/json' \
  -H 'Fidel-Key: <KEY>' \
  -d '{
    "amount": 2.55,
    "description": "Earned Stars"
  }'
```

After the reimbursement request is received by the card scheme, the newly created `id` is returned, representing the unique identifier for that reimbursement request.

```json
fileName:reimbursement.json
{
    "items": [
        {
          "id": "6c01f956-1f0f-413f-a5db-d1fc8a59ef92"
        }
    ],
    "execution": 120.856835,
    "resource": "/v1/transactions/489a79b9-92c7-4338-81ef-1529ee7bd130/reimbursement",
    "status": 202
}
```

### Lifecycle

When you request a reimbursement to an eligible cardholder transaction, we send the request to the card scheme in real time. For that reason, there is no way to stop or cancel your request after it has been sent.

Reimbursement `status` is set to `pending` while waiting for the card scheme to confirm the successful `issued` status, at which point funds normally hit the cardholder’ account. It takes between 48 to 72 hours for the `issued` status to be updated.

![Reimbursement status](https://docs.fidel.uk/assets/images/reimbursement-transactions.png "Reimbursement status")

### Status

The `status` property represents the possible reimbursement statuses:

- `pending`: scheme is executing request;

- `issued`: scheme executed request successfully;

- `failed`: scheme request failed and `error` object is created. Retry is possible. [See error list for more information](/select/reimbursement#errors).

## Reimbursement by card

In some cases, you may want to reimburse a card (and not a specific transaction). However, as card networks allow reimbursements only on specific transactions, the reimbursement request needs to be based on a transaction. The `POST https://api.fidel.uk/v1/cards/{cardId}/reimbursement` endpoint helps you to find such a transaction for this case. You can simply define the `cardId`, and Fidel API will search for the most suitable transaction, and apply the reimbursement on that one.

The request to this endpoint would require an amount and a currency (and also other optional properties):

```json
fileName: request.json
{
  "amount": 5,
  "currency": "USD",
  "description": "Earned Stars", //optional
  "brandId": "518c746f-fbf4-420a-8d37-c591adc39684" // optional
}
```

The endpoint would then respond with the unique identifier for the reimbursement that will be created for the most suitable transaction on the card.

```json
fileName: response.json
{
  "items": [
    {
      "id": "3c0c4ed5-b821-4390-8a9e-e7c140ec8358"
    }
  ],
  "resource": "/v1/cards/bc538b71-31c5-4699-840a-6d4a08693314/reimbursement",
  "status": 200,
  "execution": 19.980616
}
```

For more info, please visit the [API reference](https://reference.fidel.uk/reference/create-reimbursement-by-card) for this particular endpoint.

## Automation

Automate reimbursement requests when creating an Offer by toggling the “enable automatic reimbursements” checkbox. This toggle will be available if the reimbursement product is active and the Offer’s country is either `USA` (United States of America) or `GBR` (United Kingdom).

![Automatic reimbursement with Offers](https://docs.fidel.uk/assets/images/gifs/offers-automatic-reimbursement.gif "Automatic reimbursement with Offers")

Reimbursements will be automatically requested whenever a transaction **is both cleared and qualified** for an Offer with `offer.automatedReimbursement.enabled` set to `true`. Optionally, it's also possible to include a Description for the Reimbursement using the `offer.automatedReimbursement.description`.

Read more information on the Offers product [in the documentation page](/select/offers).

### Tracking before issuing

Track if a transaction will get a reimbursement request by checking the transaction’s `offer` object:

```json
fileName:transaction-with-offer.json
{
  // For the purpose of this example, only selected properties are shown
  "amount": 10,
  "amount": 10,
  "cleared": true,
  "currency": "USD",
  "datetime": "2021-08-20T11:11:11",
  "offer": {
    "automatedReimbursement": true,
    "id": "7e55eeae-99d6-4daf-b8c4-ac9ca660e964",
    "cashback": 20,
    "message": [],
    "performanceFee": 3.2,
    "qualified": false,
    "qualificationDate": null
  },
}
```

The `offer.automatedReimbursement` states whether a reimbursement request will be attempted for that transaction or not. The reimbursement will have the same amount as the `offer.cashback` property and the same currency as `currency`. Transactions that already have a reimbursement created for them will not have any reimbursements requested automatically.

### Tracking after issuing

Automated reimbursement attempts have the same payloads as regular reimbursements.

```json
fileName:offer-automated-reimbursement.json
{
  "accountId": "61741c3-3dc9-45f5-8e7c-db1dd649afab",
  "amount": 2.55,
  "created": "2021-09-30T11:11:11.000Z",
  "creditsTransactionId": "1250ab5a-0661-4a06-a40c-8514093a9241",
  "currency": "USD",
  "description": "Earned Stars",
  "status": "pending",
  "id": "6c01f956-1f0f-413f-a5db-d1fc8a59ef92",
  "scheme": "visa",
  "updated": "2021-09-30T11:12:11.000Z",
  "transactionId": "489a79b9-92c7-4338-81ef-1529ee7bd130",
  "programId": "4a9cef62-dc79-4044-b7aa-34425f753830"
}
```

### Failed automated reimbursements

Automated reimbursement requests can fail – e.g.: account does not have enough credits, transient issue or transaction is not reimbursement eligible (see [Eligibility](/select/reimbursement#eligibility) section). In these cases, they can be retried as described in the [Retry](/select/reimbursement#retry) section.

## Webhook

When a reimbursement status is updated from `pending` to `issued` or `failed` the webhook named `reimbursement.pending` or `reimbursement.issued` or `reimbursement.failed` is triggered (corresponding to the new status). The webhook will send the full reimbursement object with the updated `status`. See the example below with the update to `reimbursement.issued` webhook:

```json
fileName:reimbursement.json
{
  "accountId": "61741c3-3dc9-45f5-8e7c-db1dd649afab",
  "amount": 2.55,
  "created": "2021-09-30T11:11:11.000Z",
  "creditsTransactionId": "1250ab5a-0661-4a06-a40c-8514093a9241",
  "currency": "USD",
  "description": "Earned Stars",
  "status": "issued",
  "id": "6c01f956-1f0f-413f-a5db-d1fc8a59ef92",
  "scheme": "visa",
  "updated": "2021-09-30T11:12:11.000Z",
  "transactionId": "489a79b9-92c7-4338-81ef-1529ee7bd130",
  "programId": "4a9cef62-dc79-4044-b7aa-34425f753830"
}
```

## Errors

The reimbursement request and status update can fail for various reasons. Your application should handle these errors accordingly.

<h3 id="retry">Retry</h3>

Reimbursement requests can be retried via API or dashboard.

![Retry reimbursement](https://docs.fidel.uk/assets/images/gifs/retry-reimbursement.gif "Retry reimbursement")

### API Errors

These errors might be returned in the [request](/select/reimbursement/#request) endpoint.

<ReimbursementTable
headings={['HTTP Status Code', 'Error Code', 'Error Message']}
rows={[
['400', 'reimbursement-account-network-inactive', 'Account cannot issue reimbursement for card network'],
['400', 'reimbursement-account-not-enough-funds', 'Account does not have enough funds for issuing reimbursement'],
['404', 'reimbursement-account-not-found', 'Account does not exist'],
['400', 'reimbursement-already-created', 'Transaction reimbursement already created'],
['400', 'reimbursement-amount-above-limit', 'Reimbursement amount is above the allowed limit'],
['400', 'reimbursement-amount-greater-original-amount', 'Reimbursement amount is greater than original transaction amount'],
['400', 'reimbursement-invalid-transaction', 'Transaction is invalid to issue reimbursement'],
['500', 'reimbursement-network-internal-error', 'Network responded with internal error'],
['400', 'reimbursement-transaction-not-eligible', 'Transaction does not meet reimbursement eligibility criteria'],
['400', 'reimbursement-unsupported-network', 'Transaction network not supported for reimbursement'],
['400', 'reimbursement-unsupported-currency', 'Transaction currency not supported for reimbursement'],
['401', 'reimbursement-not-activated', 'The reimbursement product is not activated for this account'],
['400', 'reimbursement-visa-invalid-community-code', 'The transaction community code is invalid'],
['400', 'credits-account-not-enough-funds', 'Account does not have enough credits'],
]}
/>

### API Error Example

```json
fileName:reimbursement-request.json
{
    "error": {
        "code": "reimbursement-amount-greater-original-amount",
        "date": "2021-11-11T14:53:18.968Z",
        "message": "Reimbursement amount is greater than original transaction amount",
        "metadata": {}
    },
    "execution": 105.254359,
    "resource": "/v1/transactions/489a79b9-92c7-4338-81ef-1529ee7bd130/reimbursement",
    "status": 400
}
```

### Status Errors

These errors might be returned in the reimbursement `status` update to `failed` from the card scheme and sent to the `transaction.reimbursement.status` webhook.

<ReimbursementTable
headings={['Status Code', 'Error Code', 'Error Message']}
rows={[
['400', 'reimbursement-issuing-mismatch', 'Reimbursements issued by network do not match with original request'],
['404', 'reimbursement-network-account-not-found', 'Network was unable to find bank account'],
['404', 'reimbursement-network-customer-not-found', 'Network was unable to find customer account'],
['500', 'reimbursement-network-invalid-account', 'Network responded bank account is invalid'],
['500', 'reimbursement-network-invalid-account-country', 'Network responded bank account country is invalid'],
['500', 'reimbursement-network-invalid-currency', 'Network responded the selected currency is invalid'],
['400', 'reimbursement-network-multiple-accounts-found', 'Network responded multiple bank accounts were found'],
['500', 'reimbursement-network-others', 'Network responded with an irregular issue'],
['500', 'reimbursement-request-failed', 'Failed to submit reimbursement request to network'],
]}
/>

### Status Error Example

```json
fileName:reimbursement.json
{
  "accountId": "61741c3-3dc9-45f5-8e7c-db1dd649afab",
  "amount": 2.55,
  "created": "2021-09-30T11:11:11.000Z",
  "creditsTransactionId": "1250ab5a-0661-4a06-a40c-8514093a9241",
  "error": {
    "code": "reimbursement-network-account-not-found",
    "message": "Network was unable to find bank account",
    "status": 404
  },
  "description": "Earned Stars",
  "status": "failed",
  "id": "6c01f956-1f0f-413f-a5db-d1fc8a59ef92",
  "scheme": "visa",
  "updated": "2021-09-30T11:12:11.000Z",
  "transactionId": "489a79b9-92c7-4338-81ef-1529ee7bd130"
}
```

## API Reference

If you're looking to find out more about our Reimbursement API and how to use it with your application, please visit the [Fidel API Reference](https://reference.fidel.uk/reference).
