# A guide for verified card enrollment with the React Native SDK

Our SDK is built to also support other use cases (other than Stream). Please take the following steps to integrate and configure the SDK for your Stream application.

> Note: If an example project helps with your SDK integration & configuration, please check our [GitHub repository](https://github.com/FidelLimited/rn-sdk).

### 1. Set up your Fidel API account & your Transaction Stream program

To get started, you'll need a Fidel API account. [Sign up](https://dashboard.fidel.uk/sign-up), if you didn't create an account yet.

If you didn't create a program for your application yet, please create a Transaction Stream program from your Fidel API dashboard (or via the API).

### 2. Integrate the React Native SDK into your project

- Add the Fidel API React Native library:
    - using `npm`: 
    ```
    npm install fidel-react-native --save
    ```
    - using `yarn`:
    ```
    yarn add fidel-react-native
    ```

- In your `ios` folder, run the following command to install the Fidel dependency:

```
pod install
```

### 3. Import the SDK module

> Important note: The following SDK configuration steps need to be done as soon as the application has finished launching (usually in your App.js file)

In your project files that you want to use the SDK, please import `Fidel`:

```js
import Fidel from 'fidel-react-native';
```

### 4. Start setting up the SDK by setting the SDK Key

- Please [sign into](https://dashboard.fidel.uk/sign-in) your Fidel API dashboard account, if you didn't already.
- Click on your account name _(on the top-left hand side of the dashboard)_ -> then on `Account Settings`.
- Go to the `Plan` tab and copy your `Test` or `Live` SDK Key.
- Set your SDK Key in your app:

> Important note: For security reasons, please DO NOT store the SDK Key in your codebase. Follow our [SDK security guide](/stream/sdks/security-guidelines) for detailed recommendations.

> Note: For now, we support calling the `setup` function only once throughout the lifecycle of your app.

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey,
});
```

### 5. Add your Program ID

- Please [sign into](https://dashboard.fidel.uk/sign-in) your Fidel API dashboard account, if you didn't already.
- Go to the `Programs` section of your Fidel API dashboard.
- Click on the `Program ID` of the program that you want to enroll cards into. The program ID will be copied to your pasteboard.
- Set your Program ID in your app:

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey,
    programId: 'Your-program-ID',
});
```

### 6. Specify the program type

For your Stream application you'll need to use a Transaction Stream program, so please set:

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey,
    programId: 'Your-program-ID',
    programType: Fidel.ProgramType.transactionStream,
});
```

### 7. Configure the cardholder consent management feature

Asking for consent from the cardholder to enroll the card in your program is an important part of the SDK. The cardholder will need to read & agree with the conditions expressed using the consent language displayed during the card enrollment process. Making it clear for cardholders is essential.

You can use the `consentText` object to set the following properties:

#### Set your company name

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey,
    programId: 'Your-program-ID',
    programType: Fidel.ProgramType.transactionStream,
    consentText: {
        companyName: 'Your Company Name',
    },
});
```

#### Set your terms and conditions

```javascript
Fidel.setup ({
    ...
    consentText: {
        companyName: 'Your Company Name',
        termsAndConditionsUrl: 'https://yourwebsite.com/terms',
    },
});
```

#### Explain how the cardholder can opt out (optional, but recommended)

Please inform the cardholder about how to opt out of transaction monitoring in your program.

```javascript
Fidel.setup ({
    ...
    consentText: {
        companyName: 'Your Company Name',
        termsAndConditionsUrl: 'https://yourwebsite.com/terms',
        deleteInstructions: "how can the cardholder opt out",
    },
});
```

#### Set your privacy policy (optional, but recommended)

```javascript
Fidel.setup ({
    ...
    consentText: {
        companyName: 'Your Company Name',
        termsAndConditionsUrl: 'https://yourwebsite.com/terms',
        deleteInstructions: 'how can the cardholder opt out',
        privacyPolicyUrl: 'https://yourwebsite.com/privacy-policy',
    },
});
```

### 8. Set the allowed country(ies) of card issuance

During card enrollment process, the cardholder needs to select the country where the card was issued. Stream use cases can be activated only for cards issued in the **United States**, **Canada** and **United Kingdom**. Please configure the SDK with the following allowed countries or a subset of these countries, depending on the countries where your application is available. If the subset contains a single country, the cardholders will not need to pick the country. The country that you set will be the default country of issue for the cards that are enrolled in your program.

Use the `options` object to set the countries that you support:

```javascript
const countries = [
    Fidel.Country.canada,
    Fidel.Country.unitedKingdom,
    Fidel.Country.unitedStates,
];
Fidel.setup ({
    ...
    consentText: {
        ...
    },
    options: {
        allowedCountries: countries,
    }
});
```

### 9. Set Visa as the only supported card scheme

For Stream applications, for now, only Visa card schemes are supported. Please configure the SDK with the following property:

```javascript
Fidel.setup ({
    ...
    consentText: {
        ...
    },
    options: {
        ...
        supportedCardSchemes: [Fidel.CardScheme.visa],
    }
});
```

### 10. Check your setup

You should have the SDK set up similar to what you see in the example below:

```javascript
const countries = [
    Fidel.Country.unitedKingdom,
    Fidel.Country.unitedStates,
    Fidel.Country.canada,
];

// for now, we support calling this function only once during the lifecycle of your app
Fidel.setup({
    sdkKey: yourSdkKey, // make sure to store it securely
    programId: 'Your program ID',
    programType: Fidel.ProgramType.transactionStream,
    options: {
        allowedCountries: countries,
        supportedCardSchemes: [Fidel.CardScheme.visa],
    },
    consentText: {
        termsAndConditionsUrl: 'https://yourwebsite.com/terms',
        companyName: 'Your Company Name',
        privacyPolicyUrl: 'https://yourwebsite.com/privacy-policy',
        deleteInstructions: "how should cardholders opt out",
    }
})
```

# Verified enrollment notifications

In order to be notified about different, useful events (a card was linked, card verification started, card verification failed and others) that happen during a verified enrollment process, we recommend using our [webhooks](/stream/webhooks).

If client side notifications are useful for your application, make sure to check our SDK callbacks reference documentation.

# Enroll and verify a card

Call the following function to open the UI and start a verified card enrollment process:

```javascript
Fidel.start();
```

### Verified card enrollment flow

The following is a short description of the flow that the cardholders will experience, after calling the `start` method. You can take these steps as well to test the verified card enrollment flow, by setting a test SDK Key and by using the Fidel API [test card numbers](/stream/cards/#test-card-numbers).

If your Fidel API account is `live` then cardholders can also enroll real, live cards. Make sure that you set a live SDK Key, in order to allow live verified card enrollments.

Description of the flow:

1. An introductory screen is presented informing the cardholder about the verified enrollment flow process.
2. The card enrollment screen is presented, in which cardholders can input their card details and can also provide consent. When successful, the card will be enrolled in your Fidel API program.
3. The SDK will also immediately create a consent object for the card that is enrolled. This is what triggers the start of the verification process.
4. After this step, a card verification screen will be presented. In this screen the cardholder will be able to input the verification token (the micro-charge amount) and complete the verification process.

> Note: If the cardholders using your app do not have access to the card statement (usually, in a corporate setting), preventing them from verifying the card, you can set up the experience to involve a third-party entity (usually, a corporate card administrator) to verify the card.

### Card verification can be interrupted

After enrolling a card, its verification cannot be guaranteed to happen, for the following reasons:
- The cardholder voluntarily cancels the process after card enrollment
- The cardholder closes the app
- The system closes the app (for example, for memory management reasons, while the cardholder checks the card statement)

### How the SDK continues the card verification flow

> Note: To better visualize the process, the following diagram might be useful.

[![](https://mermaid.ink/img/pako:eNqNkctqwzAQRX9l0KqFpB_gRSGuHcgmm3QTLC8GaRwL5FHQIyXE_vcqL5p0Fa2E5pzLXHQSymkSheis-1E9-gjflWTIZ7FYvDVLo8l-HMib7viFXrfvMJ9_QkZu0B0JMbu3aXlaBUCGxBfRkJaSVbZhVUHAA-npapdnfNxSGKFqzvFwFRRG4xiC8kTcPrJrN0Ld1OydtQNxBMPRu2eyviyxvAbSH_oELf_1yEW2LnnA_R5MAE8WE6ue7o3L10s9bFo3axd7wzvoczBxaOEJeqW6mImB_IBG5286nV-kiD0NJEWRr5o6TDZKIXnKKKboNkdWoog-0UykvcZIlcGdx0EUHdpA0y8nGqQF?type=png)](https://mermaid.live/edit#pako:eNqNkctqwzAQRX9l0KqFpB_gRSGuHcgmm3QTLC8GaRwL5FHQIyXE_vcqL5p0Fa2E5pzLXHQSymkSheis-1E9-gjflWTIZ7FYvDVLo8l-HMib7viFXrfvMJ9_QkZu0B0JMbu3aXlaBUCGxBfRkJaSVbZhVUHAA-npapdnfNxSGKFqzvFwFRRG4xiC8kTcPrJrN0Ld1OydtQNxBMPRu2eyviyxvAbSH_oELf_1yEW2LnnA_R5MAE8WE6ue7o3L10s9bFo3axd7wzvoczBxaOEJeqW6mImB_IBG5286nV-kiD0NJEWRr5o6TDZKIXnKKKboNkdWoog-0UykvcZIlcGdx0EUHdpA0y8nGqQF)

If the cardholder enrolls a card, but does not complete the verification journey on the spot, your app will have to relaunch the card verification flow when the application becomes active again. The SDK will automatically attempt to complete the card verification process for you. This happens after the following app events:
- The cardholder attempts to enroll a (new) card (your app calls the `Fidel.start` method, as indicated above)
- Your application is relaunched

> Important note: To make sure that the cardholder completes the card verification process, as described above, you must configure the SDK as soon as the app launches, i.e. in your `App.js` file.

> How this is done: The SDK knows when to resume card verification, because it stores the card ID (and other useful information) securely on the device. When your application is re-launched, we check if there is an unverified card ID saved. If there is, we'll open the card verification flow.

You can also manually continue the verification flow for a specific card that was enrolled, but not verified by calling the `Fidel.verifyCard` method.

# Optional: Configure card verification executed by a third-party entity
If it is possible for the cardholders using your application to not have access to their card's statement (in a corporate setting, for example), you can configure the SDK specifically for this use case. The verification process cannot be completed by the cardholder, so a third-party who has access to the card statement needs to be involved. The third-party entity can complete the verification process to enable card data sharing.

> Note: In a setting configured for corporate cards the third party entity is usually a corporate card administrator.

### Make sure that you followed all mandatory steps to integrate and configure the SDK

Please follow all the steps written above to integrate the SDK and configure it for your application.

### Provide cardholders with the option to not verify on the spot

The following property set to `true` will give the choice to the cardholder to either verify the card on the spot or delegate it to a third-party entity. That depends on wether they have access to the card statement or not.

```javascript
Fidel.setup({
    ...
    options: {
        ...,
        thirdPartyVerificationChoice: true,
    },
})
```

### Allow a third-party entity to complete the card verification

If the cardholder has access to the card statement, verification will possibly be completed on the spot. If not, you can call the following function for the third-party entity to complete the card verification process:

```javascript
Fidel.verifyCard({
	id: 'Your card ID to verify',
	consentId: 'Your consent ID to verify',
	last4Digits: '1234',
});
```

For more details about this method, please check the reference docs of the `Fidel.verifyCard` function and its parameters.

> Important note: In order for card verification to be done successfully, make sure that you have set the `sdkKey` and `programId` properties correctly, as explained in this guide.

# Optional: Set any of the other useful properties

Please check our SDK reference page for details about any other SDK properties that might be useful for your application.

# Frequently asked questions

### How can I upgrade the React Native SDK to the latest version?

- At the root of your project, run 

`yarn upgrade fidel-react-native` or 

`npm update -g fidel-react-native`.

- In your `ios` folder, run the following command to update the Fidel dependency:
```
pod update Fidel
```

### Can I customize the UI of the SDK?

The React Native SDK offers the `options.bannerImage` property for you to set a custom, branded banner image that will be displayed during the card enrollment process. Please check our Reference documentation for more details.
