# React Native SDK reference <a style="color: #111;" class="improve-docs" href="/select/sdks/react-native/reference-v1">v1</a> <a style="margin-right: auto; border-bottom: 2px solid #0048ff;" class="improve-docs" href="/select/sdks/react-native/reference-v2">v2</a>

1. [Example](#example) with all properties set
2. [Fidel class](#class-fidel)
3. [Properties](#properties)
4. [Methods](#methods)
5. [Callbacks](#callbacks)

## Example

The following is an example set up with all properties useful for Select Transactions/Loyalty use cases:

```javascript
import Fidel, { ENROLLMENT_RESULT, ERROR } from 'fidel-react-native';
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

    const yourSupportedCountries = [
        Fidel.Country.canada,
        Fidel.Country.ireland,
        Fidel.Country.japan,
        Fidel.Country.norway,
        Fidel.Country.sweden,
        Fidel.Country.unitedArabEmirates,
        Fidel.Country.unitedKingdom, 
        Fidel.Country.unitedStates
    ];

    Fidel.setup ({
      sdkKey: yourSdkKey, // mandatory; make sure to store it securely
      programId: 'Your program ID', // mandatory
      programType: Fidel.ProgramType.transactionSelect, // optional
      options: {
        bannerImage: resolvedImage,
        allowedCountries: yourSupportedCountries,
        defaultSelectedCountry: Fidel.Country.unitedStates,
        supportedCardSchemes: [Fidel.CardScheme.visa],
        metaData: { id: 'your-metadata-id', userId: 1234 },
      },
      consentText: {
        termsAndConditionsUrl: 'https://yourwebsite.com/terms', // mandatory, for some scenarios; please check the reference below.
        companyName: 'Your Company Name', // mandatory
        privacyPolicyUrl: 'https://yourwebsite.com/privacy-policy',
        deleteInstructions: "following our delete instructions",
        programName: 'Your program name',
      },
     }, (result) => {
      switch (result.type) {
        case ENROLLMENT_RESULT:
          console.log("card was enrolled: " + result.enrollmentResult.cardId);
          break;
        case ERROR:
          this.handleError(result.error);
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

This class is designed as a facade, used to configure the card enrollment process, via many of its [static properties](#properties), and start [different flows](#methods). It's also the class that provides [callbacks](#callbacks) that might be useful for your application.

## Properties

All the properties are expected to be set once, during the lifecycle of your app, as soon as the application finished launching.

As you can see in the example above, all properties are set via the `setup` function. Some properties are grouped logically in the `consentText` and `options` child objects. In this reference we will prefix their names with their respective parent objects.

### Mandatory properties

These are properties that must be set correctly. In the case where one of these properties are not set or they are set incorrectly, the SDK will return an error in the [`main results callback`](#main-results-callback) callback (of type: `Fidel.ErrorType.sdkConfigurationError`).

#### sdkKey

Expected type: `string`

The SDK Key is used to authenticate your Fidel API account. Get it from your Fidel API dashboard -> Account Settings -> SDK Keys section.

> Important note: For security reasons, please DO NOT store the SDK Key in your codebase. Follow our [SDK security guide](/select/sdks/security-guidelines) for detailed recommendations.

> Note: If you use a **test SDK Key**, your users can only enroll [test card numbers](/select/cards/#test-card-numbers).

#### programId

Expected type: `string`

The program ID indicates the Fidel API program in which the cards will be enrolled. Get the program ID by navigating to the Fidel API dashboard -> Programs section -> Click on the ID of the program you want to use. Clicking on it will copy the ID in your pasteboard.

#### consentText.companyName

Expected type: `string`

By setting this property we customize the consent text, that the cardholder needs to read and agree with, before enrolling a card.

The maximum number of characters allowed for this property is `60`.

#### consentText.termsAndConditionsUrl

Expected type: `string`

You need to set your terms and conditions URL if you would like to:

1. support all the countries that Fidel API supports

2. set a specific `allowedCountries` set of countries AND include US or Canada in it.

By setting this property we add a link to your Terms & Conditions in the consent text. The cardholder needs to read and agree with your terms, before enrolling a card.

### Optional properties

#### options.supportedCardSchemes

Expected type: `array`

Default value: `[Fidel.CardScheme.visa, Fidel.CardScheme.mastercard, Fidel.CardScheme.americanExpress]`

Sets a list of supported card schemes. If a card scheme is supported, cardholders will be able to enroll their card. If a card scheme is not in the list, then the cardholders will see an error message while typing or pasting the unsupported card number.

If you set a `null` value, you will not be able to start the Fidel SDK enrollment flow. In this case, immediately after attempting to start the flow, you will receive an error in the [`main results callback`](#main-results-callback) callback (of type: `Fidel.ErrorType.sdkConfigurationError`).

#### options.allowedCountries

Expected type: `array`

Default value: `[Fidel.Country.canada, Fidel.Country.ireland, Fidel.Country.japan, Fidel.Country.norway, Fidel.Country.sweden, Fidel.Country.unitedArabEmirates, Fidel.Country.unitedKingdom, Fidel.Country.unitedStates]`

Sets the list of countries that cardholders can pick to be the card issuing country. When two or more countries are set, cardholders will be able to select the card issuing country with our country selection UI.

If you set a value with only one country, the country selection UI will not be displayed in the card enrollment screen. The country that you set will be considered the card issuing country for all cards enrolled in your Fidel API program using the SDK.

If you set an empty value, you will not be able to start the enrollment flow. Instead you will receive an error in the main [`callback`](#main-callback) (`Fidel.ErrorType.sdkConfigurationError`), immediately after the attempt to start.

#### options.defaultSelectedCountry

Default value: `Fidel.Country.unitedKingdom`

Sets the `Fidel.Country` that will be selected by default when the user opens the card enrollment screen. If the `defaultSelectedCountry` is not part of the `allowedCountries` list, then the first country in the `allowedCountries` list will be selected.

#### consentText.deleteInstructions

Expected type: `string`.

Default value: `'going to your account settings'`

This text informs the cardholder how to opt out of transaction monitoring in your program. It is appended at the end of the consent text. The maximum number of characters allowed for this property is `60`.

#### consentText.privacyPolicyUrl

Expected type: `string`

If you provide a value for this parameter, the card enrollment consent text will include a phrase that will provide the user with more privacy related information at the URL that you provide.

When the value of this parameter remains `nil` no such phrase will be displayed in the card enrollment consent text.

If you provide an invalid URL string, you will not be able to start the card enrollment flow. Instead you will receive an error in the [`main results callback`](#main-results-callback) callback (`Fidel.ErrorType.sdkConfigurationError`), immediately after attempting to start the card enrollment flow.

#### options.metaData

Expected type: object.

Default value: `undefined`.

This is an object that you can use to associate custom data to an enrolled card.

We advise setting an `id` value for this object. Later, it might be useful for you to use our [List Cards from Metadata ID](https://reference.fidel.uk/reference/list-cards-from-metadata-id) API Endpoint to query for cards using this ID.

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

#### consentText.programName

Expected type: `string`

Default value: `'our'`

This value is used in the consent text when enrolling a card issued in a United States or Canada.

#### programType

Default value: `Fidel.ProgramType.transactionSelect`

It specifies the type of program you want to enroll cards into. It also influences the flow that the SDK will show to cardholders when enrolling cards. The other supported value of this property is `Fidel.ProgramType.transactionStream`, but it is useful only for cards enrolled for Stream applications.

> Note: For your Loyalty application, you don't need to set this property as the default value is the correct one, for your use case.

#### options.thirdPartyVerificationChoice

Expected type: `boolean`

Default value: `false`

_Not useful for Loyalty/Select Transactions use cases, at the moment._ 

## Methods

### Fidel.setup(params, callback)

It sets up all the [properties](#properties) and [callbacks](#callbacks). This function is designed to be called only **once** during the lifecycle of your app.

Parameters:
- `params`: a JavaScript object containing all the [properties](#properties) and [callbacks](#callbacks) useful for your application.
- `callback`: the main callback in which you will receive different types of results as received during the card enrollment flow.

### Fidel.start()

Starts a card enrollment flow. If you set the `programType` to:
1. `Fidel.ProgramType.transactionStream`, a verified card enrollment flow will be started, for a Transaction Stream program (usually used by Stream applications).
2. `Fidel.ProgramType.transactionSelect`, a regular card enrollment flow will be started, for your Select Transactions program (usually used by Loyalty applications).

### Fidel.verifyCard(params)

_Not useful for Loyalty/Select Transactions use cases, at the moment._ Useful only for cards enrolled in a Transaction Stream program.

## Callbacks

The React Native SDK provides the following callbacks:

### Main results callback

Will be called when an enrollment process result is available, during the card enrollment process. It can be called multiple times with different types of results. Example:

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
      }
    });
```

#### Enrollment process results

The results are objects received in the main callback of the SDK after specific enrollment processes finish. Expect the following properties:

- `type`. This property lets you distinguish the type of result that was sent. Possible values: `ENROLLMENT_RESULT`, `VERIFICATION_RESULT`(_Not useful for Loyalty/Select transactions use cases_), `ERROR`. The possible values are constants defined in the Fidel SDK module: 

    `import Fidel, { ENROLLMENT_RESULT, ERROR } from 'fidel-react-native';`

- `enrollmentResult`. An object that is defined only when the `type` property is `ENROLLMENT_RESULT`. Please check this object's properties below.

- `error`. An object that is defined only when the `type` property is `ERROR`. Please check this object's properties below.

- `verificationResult`. _Not useful for Loyalty/Select transactions use cases_

#### Enrollment result object

A result that can be received via the main results callback, after a card is successfully enrolled in your Fidel API program.

Properties:
- `cardId`: The identifier of the card enrolled with your Fidel API program.
- `accountId`: The Fidel API account identifier.
- `programId`: The identifier of the program that the card was enrolled into.
- `enrollmentDate`: The date when the card was enrolled.
- `cardScheme`: The enrolled card's scheme.
- `isLive`: This property will be `true` when your Fidel API account is live and the card was enrolled in your `live` Fidel API program. If the program that you enrolled the card into is not a `live` one, then this property will be `false`.
- `cardFirstNumbers`: If available, this property will be populated with the first 6 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardLastNumbers`: If available, this property will be populated with the last 4 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardExpirationYear`: The expiration year of the enrolled card. The values are four digit year values (ex: 2031), **not** shortened, two digit values (ex: 31).
- `cardExpirationMonth`: The expiration month of the enrolled card. The values start with `1` (January) and end with `12` (December).
- `cardIssuingCountry`: The country where the enrolled card was issued.
- `metaData`: Custom data assigned to the enrolled card via the [`metaData`](#metadata-string-any) SDK property.

#### Error result object

An error can occur during the card enrollment process. You can handle it via the main results callback of the SDK.

Properties:

- `message`: An error message explaining more details about the error. It is not localized.
- `date`: A [timestamp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date#the_epoch_timestamps_and_invalid_date) representing when the error occurred.
- `type`: The type of the error. See more details about error types below.
- `subtype`: The subtype of the error. See more details about error subtypes below.

#### Error types

- `Fidel.ErrorType.sdkConfigurationError`: The SDK [properties](#properties) configuration is incorrect or incomplete. You can receive this error as soon as you attempt to start a flow using the SDK [methods](#methods).
- `Fidel.ErrorType.userCanceled`: The user canceled the card enrollment flow at any stage.
- `Fidel.ErrorType.deviceNotSecure`: The device that the SDK is running on is not secure (for example, when it is jailbroken/rooted).
- `Fidel.ErrorType.enrollmentError`. An error type that is received when card enrollment or consent creation fail. Check all the possible error `subtype`s below.

#### Enrollment error subtypes

- `Fidel.EnrollmentErrorType.cardAlreadyExists`: The card was already enrolled in your Fidel API program. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `Fidel.EnrollmentErrorType.invalidProgramId`: The program ID used to configure the SDK is not valid. If you receive this error, please make sure that you set a valid program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `Fidel.EnrollmentErrorType.invalidSdkKey`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `Fidel.EnrollmentErrorType.inexistentProgram`: The program ID used to configure the Fidel SDK is of a program that does not exist. If you receive this error, please make sure that you set the correct program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `item-exists`.
- `Fidel.EnrollmentErrorType.unauthorized`: The card enrollment process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `Fidel.EnrollmentErrorType.unexpected`: An unexpected error during the card enrollment step.

### onCardVerificationStarted _(!Experimental)_

_Not useful for Loyalty/Select Transactions use cases, at the moment.__Not useful for Loyalty/Select Transactions use cases, at the moment._
