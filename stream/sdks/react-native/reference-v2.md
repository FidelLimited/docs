# React Native SDK reference

1. [Example](#example) with all properties set
2. [Fidel class](#class-fidel)
3. [Properties](#properties)
4. [Methods](#methods)
5. [Callbacks](#callbacks)

## Example

The following is an example with all properties of the React Native SDK set:

```javascript
import Fidel, { ENROLLMENT_RESULT, ERROR, VERIFICATION_RESULT } from 'fidel-react-native';
//...

export default class App extends React.Component {
  constructor(props) {
    super(props);
    this.configureFidel();
  }

  configureFidel() {
    const myImage = require('./demo_images/your_banner.png');
    const resolveAssetSource = require('react-native/Libraries/Image/resolveAssetSource');
    const resolvedImage = resolveAssetSource(myImage);

    const countries = [
      Fidel.Country.unitedKingdom,
      Fidel.Country.unitedStates,
      Fidel.Country.canada,
    ];

    Fidel.setup ({
      sdkKey: yourSdkKey, // mandatory; make sure to store it securely
      programId: 'Your program ID', // mandatory
      programType: Fidel.ProgramType.transactionStream,
      options: {
        bannerImage: resolvedImage,
        allowedCountries: countries,
        supportedCardSchemes: [Fidel.CardScheme.visa],
        metaData: { id: 'your-metadata-id', userId: 1234 },
        thirdPartyVerificationChoice: false //set to true if you need to enable third party verification
      },
      consentText: {
        termsAndConditionsUrl: 'https://yourwebsite.com/terms', // mandatory
        companyName: 'Your Company Name', // mandatory
        privacyPolicyUrl: 'https://yourwebsite.com/privacy-policy',
        deleteInstructions: "following our delete instructions",
      },
      onCardVerificationStarted: consentDetails => {
        console.log('card verification started: ' + JSON.stringify(consentDetails));
      },
      onCardVerificationChoiceSelected: verificationChoice => {
        switch (verificationChoice.CardVerificationChoice) {
          case Fidel.CardVerificationChoice.onTheSpot:
            console.log('card verification choice: on the spot');
            break;
          case Fidel.CardVerificationChoice.delegatedToThirdParty:
            console.log('card verification choice: delegated to third party');
            break;
          }
        },
     }, (result) => {
      switch (result.type) {
        case ENROLLMENT_RESULT:
          console.log("card was enrolled: " + result.enrollmentResult.cardId);
          break;
        case ERROR:
          this.handleError(result.error);
          break;
        case VERIFICATION_RESULT:
          console.log('card verification was successful 🎉: ' + result.verificationResult.cardId);
          break;
      }
    });
  }

  onButtonPress = () => {
    Fidel.start();
  }

  handleError = (error) => {
    console.log("Error message: " + error.message);
    switch (error.type) {
      case Fidel.ErrorType.userCanceled:
        console.log("User canceled the process");
        break;
      case Fidel.ErrorType.sdkConfigurationError:
        console.log("Please configure the Fidel SDK correctly");
        break;
      case Fidel.ErrorType.enrollmentError:
        this.handleEnrollmentError(error);
        break;
      case Fidel.ErrorType.verificationError:
        this.handleVerificationError(error);
        break;
    }
  }

  handleEnrollmentError = (enrollmentError) => {
    switch (enrollmentError.subtype) {
      case Fidel.EnrollmentErrorType.cardAlreadyExists:
        console.log("This card was already enrolled.");
        break;
      case Fidel.EnrollmentErrorType.invalidProgramId:
        console.log("Please configure Fidel with a valid program ID.");
        break;
      case Fidel.EnrollmentErrorType.invalidSdkKey:
        console.log("Please configure Fidel with a valid SDK Key.");
        break;
      case Fidel.EnrollmentErrorType.inexistentProgram:
        console.log("Please configure Fidel with a valid program ID.");
        break;
      case Fidel.EnrollmentErrorType.unexpected:
        console.log("Unexpected enrollment error occurred.");
        break;
    }
  }

  handleVerificationError = (verificationError) => {
    switch (verificationError.subtype) {
      case Fidel.VerificationErrorType.unauthorized:
        console.log("You are not authorized to do card verification.");
        break;
      case Fidel.VerificationErrorType.incorrectAmount:
        console.log("The card verification amount entered is not correct.");
        break;
      case Fidel.VerificationErrorType.maximumAttemptsReached:
        console.log("You have reached the maximum attempts allowed to verify this card.");
        break;
      case Fidel.VerificationErrorType.cardAlreadyVerified:
        console.log("This card was already verified.");
        break;
      case Fidel.VerificationErrorType.cardNotFound:
        console.log("This card is not found.");
        break;
      case Fidel.VerificationErrorType.verificationNotFound:
        console.log("Verification not found.");
        break;
      case Fidel.VerificationErrorType.genericError:
        console.log("Generic error.");
        break;
      case Fidel.VerificationErrorType.unexpected:
        console.log("Unexpected card verification error occurred.");
        break;
    }
  }

  render() {
    return (
      <View style={styles.container}>
        <Text style={styles.welcome}>Fidel React Native SDK example</Text>
        <Text style={styles.instructions}>To get started, tap the button below.</Text>
        <Button
          onPress={this.onButtonPress}
          title="Link a card"
          color="#3846ce"
        />
      </View>
    );
  }
};
```

## class Fidel

This class is designed as a facade, used to configure the (verified) card enrollment process, via many of its [static properties](#properties), and start [different flows](#methods). It's also the class that provides [callbacks](#callbacks) that might be useful for your application.

## Properties

All the properties are expected to be set once, during the lifecycle of your app, as soon as the application finished launching.

As you can see in the example above, all properties are set via the `setup` function. Some properties are grouped logically in the `consentText` and `options` child objects. In this reference we will prefix their names with their respective parent objects.

### Mandatory properties

These are properties that must be set correctly. In the case where one of these properties are not set or they are set incorrectly, the SDK will return an error in the [`main results callback`](#main-results-callback) callback (of type: `Fidel.ErrorType.sdkConfigurationError`).

#### sdkKey

Expected type: `string`

The SDK Key is used to authenticate your Fidel API account. Get it from your Fidel API dashboard -> Account Settings -> SDK Keys section.

> Important note: For security reasons, please DO NOT store the SDK Key in your codebase. Follow our [SDK security guide](/stream/sdks/security-guidelines) for detailed recommendations.

> Note: If you use a **test SDK Key**, your users can only enroll [test card numbers](/stream/cards/#test-card-numbers).

#### programId

Expected type: `string`

The program ID indicates the Fidel API program in which the cards will be enrolled. Get the program ID by navigating to the Fidel API dashboard -> Programs section -> Click on the ID of the program you want to use. Clicking on it will copy the ID in your pasteboard.

#### consentText.companyName

Expected type: `string`

By setting this property we customize the consent text, that the cardholder needs to read and agree with, before enrolling a card.

The maximum number of characters allowed for this property is `60`.

#### consentText.termsAndConditionsUrl

Expected type: `string`

By setting this property we add a link to your Terms & Conditions in the consent text. The cardholder needs to read and agree with your terms, before enrolling a card.

### Optional properties (but we recommend setting them)

The following properties are technically not mandatory to be set. However, in order to make your Stream use case work with your Transaction Stream program, please consider setting them correctly.

#### programType

Default value: `Fidel.ProgramType.transactionSelect`

It specifies the type of program you want to enroll cards into. It also influences the flow that the SDK will show to cardholders when enrolling cards.

> Note: For your Stream application, you need to use a Transaction Stream program, so you need to set this property to `Fidel.ProgramType.transactionStream`.

#### consentText.deleteInstructions

Expected type: `string`.

Default value: `'going to your account settings'`

This text informs the cardholder how to opt out of transaction monitoring in your program. It is appended at the end of the consent text. The maximum number of characters allowed for this property is `60`.

#### consentText.privacyPolicyUrl

Expected type: `string`

If you provide a value for this parameter, the card enrollment consent text will include a phrase that will provide the user with more privacy related information at the URL that you provide.

When the value of this parameter remains `nil` no such phrase will be displayed in the card enrollment consent text.

If you provide an invalid URL string, you will not be able to start the card enrollment flow. Instead you will receive an error in the [`main results callback`](#main-results-callback) callback (`Fidel.ErrorType.sdkConfigurationError`), immediately after attempting to start the verified card enrollment flow.

#### options.supportedCardSchemes

Expected type: `array`

Default value: `[Fidel.CardScheme.visa, Fidel.CardScheme.mastercard, Fidel.CardScheme.americanExpress]`

> Important: For Stream use cases via Transaction Stream programs, only Visa cards are supported. So you need to set this property to `[Fidel.CardScheme.visa]`.

Sets a list of supported card schemes. If a card scheme is supported, cardholders will be able to enroll and verify their card. If a card scheme is not in the list, then the cardholders will see an error message while typing or pasting the unsupported card number.

If you set a `null` value, you will not be able to start the Fidel SDK verified enrollment flow. In this case, immediately after attempting to start the flow, you will receive an error in the [`main results callback`](#main-results-callback) callback (of type: `Fidel.ErrorType.sdkConfigurationError`).

#### options.allowedCountries

Expected type: `array`

Default value: `[Fidel.Country.canada, Fidel.Country.ireland, Fidel.Country.japan, Fidel.Country.norway, Fidel.Country.sweden, Fidel.Country.unitedArabEmirates, Fidel.Country.unitedKingdom, Fidel.Country.unitedStates]`

Sets the list of countries that cardholders can pick to be the card issuing country. When two or more countries are set, cardholders will be able to select the card issuing country with our country selection UI.

If you set a value with only one country, the country selection UI will not be displayed in the card enrollment screen. The country that you set will be considered the card issuing country for all cards enrolled in your Fidel API program using the SDK.

If you set an empty value, you will not be able to start the verified enrollment flow. Instead you will receive an error in the main [`callback`](#main-results-callback) (`Fidel.ErrorType.sdkConfigurationError`), immediately after the attempt to start.

> Important: For Stream use cases via Transaction Stream programs, please set only a subset of the following countries: `Fidel.Country.unitedKingdom`, `Fidel.Country.unitedStates`, `Fidel.Country.canada`, as these are the only supported countries.

#### options.defaultSelectedCountry

Default value: `Fidel.Country.unitedKingdom`

Sets the `Fidel.Country` that will be selected by default when the user opens the card enrollment screen. If the `defaultSelectedCountry` is not part of the `allowedCountries` list, then the first country in the `allowedCountries` list will be selected.

> Important: For Stream use cases via Transaction Stream programs, please use one of the following cases: `Fidel.Country.unitedKingdom`, `Fidel.Country.unitedStates`, `Fidel.Country.canada`, as these are the only supported countries.

### Optional properties

#### options.thirdPartyVerificationChoice

Expected type: `boolean`

Default value: `false`

When set to `true`, after enrolling the card, the cardholder will enter a screen from which it is possible to choose to continue the experience based on whether the cardholder:
1. Has access to the card statement. In this case the experience will continue with the card verification screen allowing the cardholder to complete the process on the spot.
2. Does not have access to the card statement. In this case the verification of the card will not be done by the cardholder. It needs to be delegated to a third party entity. Usually, in a corporate card setting, the third party entity is a corporate card administrator.

#### options.metaData

Expected type: object.

Default value: `null`.

This is an object that you can use to associate custom data to an enrolled card.

We advise setting an `id` value for this object. Later, it might be useful for you to use our [List Cards from Metadata ID](https://transaction-stream.fidel.uk/reference/list-cards-from-metadata-id) API Endpoint to query for cards using this ID.

Example of meta data that you can set:

```javascript
metaData: {
    id: "this-is-the-metadata-id",
    userId: "123",
    customKey: 456
}
```

You would receive an object equal to this one, after successfully enrolling a card, in the main callback of the SDK.

#### options.bannerImage

Will display the banner image that you set in this parameter at the top of the card details screen. Your custom asset needs to be resolved in order to be passed to our native module:

```javascript
const myImage = require('./images/your_banner.png');
const resolveAssetSource = require('react-native/Libraries/Image/resolveAssetSource');
const resolvedImage = resolveAssetSource(myImage);
```

The banner image will take the device's width, but it has a fixed height of 100 pts.
The image view has an `Aspect Fill` content mode, which means that the banner image that you set will fill its entire predefined area, while keeping the aspect ratio.

For the banner image that you can set, we suggest to use the aspect ratio of the smallest devices that you support. On wider devices, the banner image will be cropped from top and bottom sides. This is because of the `Aspect Fill` content mode that we set for the image view.

If a device that opens the SDK has 320dp in width, the aspect ratio of the image view would be *320 : 100*.
If a device that opens the SDK has 475dp in width, the aspect ratio of your banner image would be *475 : 100*.

You need to provide the image for all screen densities (x1, x2 and x3).

Depending on what you want to display in the banner image, you might need to experiment a bit to make sure that nothing important from the image is hidden. The most important information should be displayed in the centre of the banner image.


### Properties that are not in use for Transaction Stream programs

The following properties are only useful when enrolling cards in a Select Transactions program.

#### consentText.programName

Expected type: `string`

Default value: `'our'`

This value is used in the consent text when enrolling a card issued in a United States or Canada, in a Select Transactions program.

## Methods

### Fidel.setup(params, callback)

It sets up all the [properties](#properties) and [callbacks](#callbacks). This function is designed to be called only **once** during the lifecycle of your app.

Parameters:
- `params`: a JavaScript object containing all the [properties](#properties) and [callbacks](#callbacks) useful for your application.
- `callback`: the main callback in which you will receive different types of results as received during the card enrollment flow.

### Fidel.start()

Starts a card enrollment flow. If you set the `programType` to:
1. `Fidel.ProgramType.transactionStream`, a verified card enrollment flow will be started, for your Transaction Stream program (usually used by Stream applications).
2. `Fidel.ProgramType.transactionSelect`, a regular card enrollment flow will be started, for your Transaction Select program (usually used by Loyalty applications).

### Fidel.verifyCard(params)

Starts a card verification flow.

#### Parameters

- `params`: an object specifying the card verification configuration, used to start the card verification flow. This is a *mandatory* parameter. Expected properties of this object:

    - `id` - The card identifier for which verification will be performed. Expected type: `string`.
    - `consentId` - The card consent identifier used for card verification. Expected type: `string`.
    - `last4Digits` - The last 4 digits of a card. If set, the verification screen will display them to visually help the the card verifier identify which card is being verified. If not set, the verification screen will be more generic. Expected type: `string`. This property is *optional*.

## Callbacks

The React Native SDK provides the following callbacks:

### Main results callback

Will be called when an enrollment process result is available, during the verified card enrollment process. It can be called multiple times with different types of results. Example:

```javascript
Fidel.setup ({
      sdkKey: yourSdkKey, // make sure to store it securely
      programId: 'Your program ID',
      // other properties ...
     }, (result) => {
      switch (result.type) {
        case ENROLLMENT_RESULT:
          console.log("card was enrolled: " + result.enrollmentResult.cardId);
          break;
        case ERROR:
          this.handleError(result.error);
          break;
        case VERIFICATION_RESULT:
          console.log('card verification was successful 🎉: ' + result.verificationResult.cardId);
          break;
      }
    });
```

#### Enrollment process results

The results are objects received in the main callback of the SDK after specific enrollment processes finish. Expect the following properties:

- `type`. This property lets you distinguish the type of result that was sent. Possible values: `ENROLLMENT_RESULT`, `VERIFICATION_RESULT`, `ERROR`. The possible values are constants defined in the Fidel SDK module: 

    `import Fidel, { ENROLLMENT_RESULT, ERROR, VERIFICATION_RESULT } from 'fidel-react-native';`

- `enrollmentResult`. An object that is defined only when the `type` property is `ENROLLMENT_RESULT`. Please check this object's properties below.

- `verificationResult`. An object that is defined only when the `type` property is `VERIFICATION_RESULT`. The only property of this object is `cardId`, representing the card identifier for which verification was completed.

- `error`. An object that is defined only when the `type` property is `ERROR`. Please check this object's properties below.

#### Enrollment result object

A result that can be received via the main results callback, after a card is successfully enrolled in your Fidel API program.

Properties:
- `cardId`: The identifier of the card enrolled with your Fidel API program.
- `accountId`: The Fidel API account identifier.
- `programId`: The identifier of the program that the card was enrolled into.
- `enrollmentDate`: The date when the card was enrolled.
- `cardScheme`: The enrolled card's scheme. For Transaction Stream programs, for now, the only possible value is `Fidel.CardScheme.visa`.
- `isLive`: This property will be `true` when your Fidel API account is live and the card was enrolled in your `live` Fidel API program. If the program that you enrolled the card into is not a `live` one, then this property will be `false`.
- `cardFirstNumbers`: If available, this property will be populated with the first 6 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardLastNumbers`: If available, this property will be populated with the last 4 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardExpirationYear`: The expiration year of the enrolled card. The values are four digit year values (ex: 2031), **not** shortened, two digit values (ex: 31).
- `cardExpirationMonth`: The expiration month of the enrolled card. The values start with `1` (January) and end with `12` (December).
- `cardIssuingCountry`: The country where the enrolled card was issued.
- `metaData`: Custom data assigned to the enrolled card via the [`metaData`](#metadata-string-any) SDK property.

#### Card verification result object

A result that can be received via the main result callback, after a card is successfully verified.

Properties:

- `cardId`: The identifier of the card that was successfully verified.

#### Error result object

An error can occur during the card enrollment, card consent creation or card verification processes. You can handle it via the main results callback of the SDK.

Properties:

- `message`: An error message explaining more details about the error. It is not localized.
- `date`: A [timestamp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date#the_epoch_timestamps_and_invalid_date) representing when the error occurred.
- `type`: The type of the error. See more details about error types below.
- `subtype`: The subtype of the error. See more details about error subtypes below.

#### Error types

- `Fidel.ErrorType.sdkConfigurationError`: The SDK [properties](#properties) configuration is incorrect or incomplete. You can receive this error as soon as you attempt to start a flow using the SDK [methods](#methods).
- `Fidel.ErrorType.userCanceled`: The user canceled the verified card enrollment flow at any stage.
- `Fidel.ErrorType.deviceNotSecure`: The device that the SDK is running on is not secure (for example, when it is jailbroken/rooted).
- `Fidel.ErrorType.enrollmentError`. An error type that is received when card enrollment or consent creation fail. Check all the possible error `subtype`s below.
- `Fidel.ErrorType.verificationError`. An error type that is received when card verification fails. Check all the possible errors: Check all the possible card verification error `subtype`s below.

#### Enrollment error subtypes

- `Fidel.EnrollmentErrorType.cardAlreadyExists`: The card was already enrolled in your Fidel API program. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `Fidel.EnrollmentErrorType.invalidProgramId`: The program ID used to configure the SDK is not valid. If you receive this error, please make sure that you set a valid program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `Fidel.EnrollmentErrorType.invalidSdkKey`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `Fidel.EnrollmentErrorType.inexistentProgram`: The program ID used to configure the Fidel SDK is of a program that does not exist. If you receive this error, please make sure that you set the correct program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `item-exists`.
- `Fidel.EnrollmentErrorType.unauthorized`: The card enrollment process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `Fidel.EnrollmentErrorType.issuerProcessingError`: When starting the card verification process, there was an issue encountered by the card issuer when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-issuer-processing-charge`.
- `Fidel.EnrollmentErrorType.duplicateTransactionError`: When starting the card verification process, a duplicated transaction encountered when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-duplicate-card-transaction`.
- `Fidel.EnrollmentErrorType.insufficientFundsError`: When starting the card verification process, insufficient funds when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-insufficient-card-funds`.
- `Fidel.EnrollmentErrorType.processingChargeError`: When starting the card verification process, there was an error processing the micro-charge. It is possible to retry later. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-processing-charge`.
- `Fidel.EnrollmentErrorType.cardDetailsError`: When starting the card verification process, an error related to the card details was found, when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-incorrect-card-details`.
- `Fidel.EnrollmentErrorType.cardLimitExceededError`: When starting the card verification process, the enrolled card has exceeded its limit for transactions when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-card-limit-exceeded`.
- `Fidel.EnrollmentErrorType.unexpected`: An unexpected error during the card enrollment step.

#### Card verification error subtypes

- `Fidel.VerificationErrorType.invalidSDKKey`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `Fidel.VerificationErrorType.incorrectAmount`: The verification token is incorrect and the card cannot be verified. This error is equivalent to the Fidel API error with the code `verification-incorrect-amount`.
- `Fidel.VerificationErrorType.incorrectAmountCode`: The verification token is incorrect and the card cannot be verified. This error is received when verifying card consents using the Consents API. This error is equivalent to the Fidel API error with the code `card-consent-authentication-code-incorrect`.
- `Fidel.VerificationErrorType.maximumAttemptsReached`: As a security measure, we allow only a limited amount of attempts to verify a card. This error is received when the cardholder reaches the maximum number of attempts to verify the card. This error is equivalent to the Fidel API error with the code `verification-max-attempts-reached`.
- `Fidel.VerificationErrorType.cardAlreadyVerified`: The card was already verified, perhaps from another device or through another method. This error is equivalent to the Fidel API error with the code `card-already-verified`.
- `Fidel.VerificationErrorType.cardNotFound`: Error received when the card that was attempted to be verified was not 
found. This error is equivalent to the Fidel API error with the code `card-not-found`.
- `Fidel.VerificationErrorType.verificationNotFound`: This error is equivalent to the Fidel API error with the code 
`verification-not-found`.
- `Fidel.VerificationErrorType.genericError`: This error is equivalent to the Fidel API error with the code `verification-error-generic`.
- `Fidel.VerificationErrorType.unauthorized`: The card verification process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `Fidel.VerificationErrorType.unexpected`: An unexpected error during the card verification step.

### onCardVerificationStarted

A callback set via the `setup` function's `params`. Will be called when the card verification process has started. This callback is similar to the `card.verification.started` webhook that Fidel API is providing. We advise using the webhook, in most situations, to understand when card verification has started for a card that was enrolled. Example of this callback:

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey, // make sure to store it securely
    programId: 'Your program ID',
    // ... other properties
    onCardVerificationStarted: consentDetails => {
        console.log('card verification started: ' + JSON.stringify(consentDetails));
    },
})
```

Parameters:
- a object representing the details of the consent that the SDK creates after enrolling the card. This is what triggers the verification process to start. Properties of this object:
    - `cardId: String`: The card ID for which the consent was created.
    - `consentId: String`: The identifier of the consent that was created.
    - `programId: String`: The identifier of the program the card was enrolled into.

### onCardVerificationChoiceSelected _(!Experimental)_

This is an _experimental_ feature. This callback is set via the `setup` function's `params`. It will be called when you set `thirdPartyVerificationChoice` property to `true` and the cardholder makes a card verification choice. Example of this callback:

```javascript
Fidel.setup ({
    sdkKey: yourSdkKey, // make sure to store it securely
    programId: 'Your program ID',
    // ... other properties
    onCardVerificationChoiceSelected: verificationChoice => {
        switch (verificationChoice.CardVerificationChoice) {
            case Fidel.CardVerificationChoice.onTheSpot:
                console.log('card verification choice: on the spot');
                break;
            case Fidel.CardVerificationChoice.delegatedToThirdParty:
                console.log('card verification choice: delegated to third party');
                break;
        }
    },
})
```

The parameter is a card verification choice, as expressed by the cardholder. Possible values:
- `Fidel.CardVerificationChoice.onTheSpot`: It means that the cardholder acknowledges that the verification token can be accessed and card verification can be done on the spot.
- `Fidel.CardVerificationChoice.delegatedToThirdParty`: The cardholder acknowledges that the verification token cannot be accessed and card verification cannot be done on the spot. A third party entity must be involved to verify the card.
