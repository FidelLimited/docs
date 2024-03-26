# iOS SDK reference

1. [Fidel class](#class-fidel)
2. [Properties](#properties)
3. [Methods](#methods)
4. [Callbacks](#callbacks)

## class Fidel

This class is designed as a facade, used to configure the (verified) card enrollment process, via many of its [static properties](#properties), and start [different flows](#methods). It's also the class that provides [callbacks](#callbacks) that might be useful for your application.

## Properties

### Mandatory properties

These are properties that must be set correctly. In the case where one of these properties are not set or they are set incorrectly, the SDK will return an error in the `onResult` callback (of type: `FidelErrorType.sdkConfigurationError`).

#### sdkKey: String

This key is used to authenticate your Fidel API account. Get it from your Fidel API dashboard -> Account Settings -> SDK Keys section.

> Important note: For security reasons, please DO NOT store the SDK Key in your codebase. Follow our [SDK security guide](/stream/sdks/security-guidelines) for detailed recommendations.

> Note: If you use a **test SDK Key**, your users can only enroll [test card numbers](/stream/cards/#test-card-numbers).

#### programID: String

The program ID indicates the Fidel API program in which the cards will be enrolled. Get the program ID by navigating to the Fidel API dashboard -> Programs section -> Click on the ID of the program you want to use. Clicking on it will copy the ID in your pasteboard.

#### companyName: String

By setting this property we customize the consent text, that the cardholder needs to read and agree with, before enrolling a card.

The maximum number of characters allowed for this property is `60`.

#### termsAndConditionsURL: String

By setting this property we add a link to your Terms & Conditions in the consent text. The cardholder needs to read and agree with your terms, before enrolling a card.

### Optional properties (but we recommend setting them)

The following properties are technically not mandatory to be set. However, in order to make your Stream use case work with your Transaction Stream program, please consider setting them correctly.

#### programType: ProgramType

Default value: `.transactionSelect`

It specifies the type of program you want to enroll cards into. The `ProgramType` influences the flow that the SDK will show to cardholders when enrolling cards. 

> Note: For your Stream application, you need to use a Transaction Stream program, so you need to set this property to `.transactionStream`.

#### supportedCardSchemes

Type: `Set<CardScheme>`

Default value: `[.visa, .mastercard, .americanExpress]`

> Important: For Stream use cases via Transaction Stream programs, only Visa cards are supported. So you need to set this property to `[.visa]`.

Sets a list of supported card schemes. If a card scheme is supported, cardholders will be able to enroll and verify their card. If a card scheme is not in the list, then the cardholders will see an error message while typing or pasting the unsupported card number.

If you set a `nil` value, you will not be able to start the Fidel SDK verified enrollment flow. In this case, immediately after attempting to start the flow, you will receive an error in the `onResult` callback (of type: `FidelErrorType.sdkConfigurationError`).

#### enum CardScheme

Cases:

- `visa`, `mastercard`, `americanExpress`

#### allowedCountries

Type: `Set<Country>`

Default value: `[.canada, .ireland, .japan, .unitedKingdom, .unitedStates, .sweden, .unitedArabEmirates, .norway]`

> Important: For Stream use cases via Transaction Stream programs, please set only a subset of the following countries: `.unitedKingdom`, `.unitedStates`, `.canada`, as these are the only supported countries.

Sets the list of countries that cardholders can pick to be the card issuing country. When two or more countries are set, cardholders will be able to select the card issuing country with our country selection UI.

If you set a value with only one country (`count` of this set == 1), the country selection UI will not be displayed in the card enrollment screen. The country that you set will be considered the card issuing country for all cards enrolled in your Fidel API program using the SDK.

If you set an empty value, you will not be able to start the verified enrollment flow. Instead you will receive an error in the `onResult` callback (`FidelErrorType.sdkConfigurationError`), immediately after the attempt to start.

See more: [Country](#enum-country)

#### enum Country

Cases:

- `canada`, `ireland`, `japan`, `norway`, `sweden`, `unitedArabEmirates`, `unitedKingdom`, `unitedStates`

#### deleteInstructions: String

Default value: `"going to your account settings"`

This text informs the cardholder how to opt out of transaction monitoring in your program. It is appended at the end of the consent text. The maximum number of characters allowed for this property is `60`.

#### defaultSelectedCountry: Country

Default value: `.unitedKingdom`

> Important: For Stream use cases via Transaction Stream programs, please use one of the following cases: `.unitedKingdom`, `.unitedStates`, `.canada`, as these are the only supported countries.

Sets the `Country` that will be selected by default when the user opens the card enrollment screen. If the `defaultSelectedCountry` is not part of the `allowedCountries` list, then the first country in the `allowedCountries` list will be selected.

#### privacyPolicyURL: String?

Default value: `nil`.

If you provide a value for this parameter, the card enrollment consent text will include a phrase that will provide the user with more privacy related information at the URL that you provide.

When the value of this parameter remains `nil` no such phrase will be displayed in the card enrolling consent text.

If you provide an invalid URL string, you will not be able to start the card enrollment flow. Instead you will receive an error in the `onResult` callback (`FidelErrorType.sdkConfigurationError`), immediately after attempting to start the verified card enrollment flow.

### Optional properties

#### thirdPartyVerificationChoice: Bool

Default value: `false`.

When set to `true`, after enrolling the card, the cardholder will enter a screen from which it is possible to choose to continue the experience based on whether the cardholder:
1. Has access to the card statement. In this case the experience will continue with the card verification screen allowing the cardholder to complete the process on the spot.
2. Does not have access to the card statement. In this case the verification of the card will not be done by the cardholder. It needs to be delegated to a third party entity. Usually, in a corporate card setting, the third party entity is a corporate card administrator.

#### metaData: [String: Any]?

This is a dictionary that you can use to associate custom data with an enrolled card.

We advise setting an `"id"` key -> value pair in this dictionary. Later, it might be useful for you to use our [List Cards from Metadata ID](https://transaction-stream.fidel.uk/reference/list-cards-from-metadata-id) API Endpoint to query for cards using this ID.

Example of meta data that you can set:

```swift
Fidel.metaData = [
    "id": "this-is-the-metadata-id",
    "myUserId": "123",
    "customKey1": "customValue1"
]
```

You would receive a dictionary equal to this one, after successfully enrolling a card, in the `onResult` callback, in the `EnrollmentResult` object.

#### bannerImage: UIImage?

Default value: `nil`.

Will display the banner image that you set in this parameter at the top of the card details screen.

The banner image will take the device's width, but it has a fixed height of 100 pts.
The image view has an `Aspect Fill` content mode, which means that the banner image that you set will fill its entire predefined area, while keeping the aspect ratio.

For the banner image that you can set, we suggest to use the aspect ratio of the smallest devices that you support. On wider devices, the banner image will be cropped from top and bottom sides. This is because of the `Aspect Fill` content mode that we set for the image view.

If you support 4" iPhones (iPhone 5s, 5c etc.), the aspect ratio would be *320 : 100*.
If the smallest device that you support is 4.7" iPhones (iPhone 6, 7, 8 etc.),
the aspect ratio of your banner image would be *375 : 100*.

You need to provide the image for all screen densities (x1, x2 and x3).

Depending on what you want to display in the banner image, you might need to experiment a bit to make sure that nothing important from the image is hidden. The most important information should be displayed in the centre of the banner image.


### Properties that are not in use for Transaction Stream programs

The following properties are only useful when enrolling cards in a Select Transactions program.

#### programName: String

Default value: `"our"`

This value is used in the consent text when enrolling a card issued in a United States or Canada, in a Select Transactions program.

## Methods

### start(from:)

Starts a card enrollment flow. If you set the `programType` to:
1. `.transactionStream`, a verified card enrollment flow will be started, for your Transaction Stream program (usually used by Stream applications).
2. `.transactionSelect`, a regular card enrollment flow will be started, for your Transaction Select program (usually used by Loyalty applications).

#### Parameters
- `startingViewController: UIViewController`: the `UIViewController` that will start the card enrollment flow. This is a *mandatory* parameter.

### verifyCard(from:cardVerificationConfiguration:)

Starts a card verification flow.

#### Parameters

- `startingViewController: UIViewController`: the `UIViewController` that will start the Fidel SDK card verification flow. This is a *mandatory* parameter.
- `cardVerificationConfiguration: CardVerificationConfiguration`: the configuration, as defined below, that should be used to start the Fidel SDK card verification flow. This is a *mandatory* parameter.

##### struct CardVerificationConfiguration

It represents the configuration that you need to prepare in order to start a card verification flow.

Properties:

- `cardID: String` - The card identifier for which verification will be performed.
- `consentID: String` - The card consent identifier used for card verification.
- `last4Digits: String?` - The last 4 digits of a card. If set, the verification screen will display them to visually help the the card verifier identify which card is being verified. If not set, the verification screen will be more generic. This property is *optional*.

## Callbacks

The iOS SDK provides the following callbacks:

### onResult

Will be called when a `FidelResult` is available, during the verified card enrollment process. It can be called multiple times with different types of results.

#### enum FidelResult

Cases:

- `enrollmentResult(EnrollmentResult)`: Received after successfully enrolling the card in your Fidel API program (the card is not yet verified). Please check more details about `EnrollmentResult` below.
- `verificationResult(VerificationResult)`. Received after successfully verifying the card that was enrolled in your Fidel API program. Please check more details about `VerificationResult` below.
- `error(FidelError)`. An error occurred either during the card enrollment, card consent creation or card verification processes. Please check more details about `FidelError` below.

#### struct EnrollmentResult

A result that can be received via the `onResult` callback, after a card is successfully enrolled in your Fidel API program.

Properties:
- `cardID: String`: The identifier of the card enrolled with your Fidel API program.
- `accountID: String`: The Fidel API account identifier.
- `programID: String`: The identifier of the program that the card was enrolled into.
- `enrollmentDate: Date`: The date when the card was enrolled.
- `scheme: CardScheme`: The enrolled card's scheme.
- `isLive: Bool`: This property will be `true` when your Fidel API account is live and the card was enrolled in your `live` Fidel API program. If the program that you enrolled the card into is not a `live` one, then this property will be `false`.
- `cardFirstNumbers: String?`: If available, this property will be populated with the first 6 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardLastNumbers: String?`: If available, this property will be populated with the last 4 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardExpirationYear: Int`: The expiration year of the enrolled card. The values are four digit year values (ex: 2031), **not** shortened, two digit values (ex: 31).
- `cardExpirationMonth: Int`: The expiration month of the enrolled card. The values start with `1` (January) and end with `12` (December).
- `cardIssuingCountry: Country`: The country where the enrolled card was issued.
- `metaData: [String: Any]?`: Custom data assigned to the enrolled card via the `metaData` SDK property.

#### struct VerificationResult

A result that can be received via the `onResult` callback, after a card is successfully verified.

Properties:

- `cardID: String`: The identifier of the card that was successfully verified.

#### struct FidelError

A FidelError can occur during the card enrollment, card consent creation or card verification processes. You can handle it via the `onResult` callback.

Properties:

- `message: String`: An error message explaining more details about the error. It is not localized.
- `date: Date`: When the error occurred.
- `type: FidelErrorType`: The type of the error. Please check more details about `FidelErrorType` below.

#### enum FidelErrorType

Cases:

- `sdkConfigurationError`: The SDK [properties](#properties) configuration is incorrect or incomplete. You can receive this error as soon as you attempt to start a flow using the SDK [methods](#methods).
- `userCanceled`: The user canceled the verified card enrollment flow at any stage.
- `deviceNotSecure`: The device that the SDK is running on is not secure (for example, when it is jailbroken).
- `enrollmentError(EnrollmentError)`. An error that is received during card enrollment or during consent creation. Please check more details about `EnrollmentError` below.
- `verificationError(VerificationError)`. An error that occurs when card verification fails. Please check more details about `VerificationError` below.

#### enum EnrollmentError

An enum containing some of the cases of card enrollment errors.

Cases:

- `cardAlreadyExists`: The card was already enrolled in your Fidel API program. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `invalidProgramID`: The program ID used to configure the SDK is not valid. If you receive this error, please make sure that you set a valid program ID via the `programID` property. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `invalidSDKKey`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `inexistentProgram`: The program ID used to configure the Fidel SDK is of a program that does not exist. If you receive this error, please make sure that you set the correct program ID via the `programID` property. This error is equivalent to the Fidel API error with the code `item-exists`.
- `unauthorized`: The card enrollment process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `issuerProcessingError`: When starting the card verification process, there was an issue encountered by the card issuer when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-issuer-processing-charge`.
- `duplicateTransactionError`: When starting the card verification process, a duplicated transaction encountered when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-duplicate-card-transaction`.
- `insufficientFundsError`: When starting the card verification process, insufficient funds when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-insufficient-card-funds`.
- `processingChargeError`: When starting the card verification process, there was an error processing the micro-charge. It is possible to retry later. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-processing-charge`.
- `cardDetailsError`: When starting the card verification process, an error related to the card details was found, when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-incorrect-card-details`.
- `cardLimitExceededError`: When starting the card verification process, the enrolled card has exceeded its limit for transactions when processing the micro-charge. This error is equivalent to the Fidel API Create Consent error with the code `card-consent-card-limit-exceeded`.
- `unexpected`: An unexpected error during the card enrollment step.

#### enum VerificationError

An enum containing some of the cases of card verification errors.

Cases:

- `invalidSDKKey`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `incorrectAmount`: The verification token is incorrect and the card cannot be verified. This error is equivalent to the Fidel API error with the code `verification-incorrect-amount`.
- `incorrectAmountCode`: The verification token is incorrect and the card cannot be verified. This error is received when verifying card consents using the Consents API. This error is equivalent to the Fidel API error with the code `card-consent-authentication-code-incorrect`.
- `maximumAttemptsReached`: As a security measure, we allow only a limited amount of attempts to verify a card. This error is received when the cardholder reaches the maximum number of attempts to verify the card. This error is equivalent to the Fidel API error with the code `verification-max-attempts-reached`.
- `cardAlreadyVerified`: The card was already verified, perhaps from another device or through another method. This error is equivalent to the Fidel API error with the code `card-already-verified`.
- `cardNotFound`: Error received when the card that was attempted to be verified was not 
found. This error is equivalent to the Fidel API error with the code `card-not-found`.
- `verificationNotFound`: This error is equivalent to the Fidel API error with the code 
`verification-not-found`.
- `genericError`: This error is equivalent to the Fidel API error with the code `verification-error-generic`.
- `unauthorized`: The card verification process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `unexpected`: An unexpected error during the card verification step.

### onCardVerificationStarted

Will be called when the card verification process has started. This callback is similar to the `card.verification.started` webhook that Fidel API is providing. We advise using the webhook, in most situations, to understand when card verification has started for a card that was enrolled.

Parameters:
- a `ConsentDetails` object, representing the details of the consent that the SDK creates after enrolling the card. This is what triggers the verification process to start. See more details below related to `ConsentDetails`.

#### struct ConsentDetails

The details of the consent that the SDK creates via the Fidel API in order to start a card verification process.

Properties:
- `cardID: String`: The card ID for which the consent was created.
- `consentID: String`: The identifier of the consent that was created.
- `programID: String`: The identifier of the program the card was enrolled into.

### onCardVerificationChoiceSelected _(!Experimental)_

This is an _experimental_ feature. It will be called when you set `thirdPartyVerificationChoice` property to `true` and the cardholder makes a card verification choice.

Parameters:
 - a `CardVerificationChoice` case.

 #### enum CardVerificationChoice _(!Experimental)_

A card verification choice expressed by the cardholder, when the third party verification choice is enabled.

Cases:
- `onTheSpot`: It means that the cardholder acknowledges that the verification token can be accessed and card verification can be done on the spot.
- `delegatedToThirdParty`: The cardholder acknowledges that the verification token cannot be accessed and card verification cannot be done on the spot. A third party entity must be involved to verify the card.
