# Android SDK reference <a style="color: #111;" class="improve-docs" href="/select/sdks/android/reference-v1">v1</a> <a style="margin-right: auto; border-bottom: 2px solid #0048ff;" class="improve-docs" href="/select/sdks/android/reference-v2">v2</a>

1. [Fidel class](#class-fidel)
2. [Properties](#properties)
3. [Methods](#methods)
4. [Callbacks](#callbacks)

## class Fidel

This class is designed as a facade, used to configure the card enrollment process, via many of its [static properties](#properties), and start the [card enrollment flow](#methods). It's also the class that provides [callbacks](#callbacks) that might be useful for your application.

## Properties

### Mandatory properties

These are properties that must be set correctly. In the case where one of these properties are not set or they are set incorrectly, the SDK will return an error in the `onResult` callback (of type: `FidelErrorType.SdkConfigurationError`).

#### sdkKey: String

This key is used to authenticate your Fidel API account. Get it from your Fidel API dashboard -> Account Settings -> SDK Keys section.

> Important note: For security reasons, please DO NOT store the SDK Key in your codebase. Follow our [SDK security guide](/select/sdks/security-guidelines) for detailed recommendations.

> Note: If you use a **test SDK Key**, your users can only enroll [test card numbers](/select/cards/#test-card-numbers).

#### programId: String

The program ID indicates the Fidel API program in which the cards will be enrolled. Get the program ID by navigating to the Fidel API dashboard -> Programs section -> Click on the ID of the program you want to use. Clicking on it will copy the ID in your pasteboard.

#### companyName: String

By setting this property we customize the consent text, that the cardholder needs to read and agree with, before enrolling a card.

The maximum number of characters allowed for this property is `60`.

#### termsAndConditionsUrl: String

You need to set your terms and conditions URL if you would like to:

1. support all the countries that Fidel API supports

2. set a specific `allowedCountries` set of countries AND include US or Canada in it.

By setting this property we add a link to your Terms & Conditions in the consent text. The cardholder needs to read and agree with your terms, before enrolling a card.

### Optional properties

#### supportedCardSchemes

Type: `Set<CardScheme>`

Default value: `setOf(CardScheme.VISA, CardScheme.MASTERCARD, CardScheme.AMERICAN_EXPRESS)`

Sets a list of supported card schemes. If a card scheme is supported, cardholders will be able to enroll their card. If a card scheme is not in the list, then the cardholders will see an error message while typing or pasting the unsupported card number.

If you set a `null` value, you will not be able to start the Fidel SDK enrollment flow. In this case, immediately after attempting to start the flow, you will receive an error in the `onResult` callback (of type: `FidelErrorType.SdkConfigurationError`).

#### enum class CardScheme

Cases:

- `VISA`, `MASTERCARD`, `AMERICAN_EXPRESS`

#### allowedCountries

Type: `Set<Country>`

Default value: `setOf(Country.CANADA, Country.IRELAND, Country.JAPAN, Country.NORWAY, Country.SWEDEN, Country.UNITED_ARAB_EMIRATES, Country.UNITED_KINGDOM, Country.UNITED_STATES)`

Sets the list of countries that cardholders can pick to be the card issuing country. When two or more countries are set, cardholders will be able to select the card issuing country with our country selection UI.

If you set a value with only one country (`size` of this set == 1), the country selection UI will not be displayed in the card enrollment screen. The country that you set will be considered the card issuing country for all cards enrolled in your Fidel API program using the SDK.

If you set an empty value, you will not be able to start the enrollment flow. Instead you will receive an error in the `onResult` callback (`FidelErrorType.SdkConfigurationError`), immediately after the attempt to start.

##### enum class Country

Cases:

- `CANADA, IRELAND, JAPAN, NORWAY, SWEDEN, UNITED_ARAB_EMIRATES, UNITED_KINGDOM, UNITED_STATES`

#### deleteInstructions: String

Default value: `"going to your account settings"`

This text informs the cardholder how to opt out of transaction monitoring in your program. It is appended at the end of the consent text. The maximum number of characters allowed for this property is `60`.

#### defaultSelectedCountry: Country

Default value: `UNITED_KINGDOM`

Sets the `Country` that will be selected by default when the user opens the card enrollment screen. If the `defaultSelectedCountry` is not part of the `allowedCountries` list, then the first country in the `allowedCountries` list will be selected.

#### privacyPolicyUrl: String?

Default value: `null`.

If you provide a value for this parameter, the card enrollment consent text will include a phrase that will provide the user with more privacy related information at the URL that you provide.

When the value of this parameter remains `null` no such phrase will be displayed in the card enrolling consent text.

If you provide an invalid URL string, you will not be able to start the card enrollment flow. Instead you will receive an error in the `onResult` callback (`FidelErrorType.SdkConfigurationError`), immediately after attempting to start the card enrollment flow.

#### metaData: org.json.JSONObject?

This is a JSONObject that you can use to associate custom data with an enrolled card.

We advise setting an `"id"` key -> value pair in this JSONObject. Later, it might be useful for you to use our [List Cards from Metadata ID](https://reference.fidel.uk/reference/list-cards-from-metadata-id) API Endpoint to query for cards using this ID.

Example of meta data that you can set:

```swift
val jsonMetaData = JSONObject()
jsonMetaData.put("id", "this-is-the-metadata-id")
jsonMetaData.put("myUserId", "123")
jsonMetaData.put("customKey", "customValue")
Fidel.metaData = jsonMetaData
```

You would receive a JSONObject that is equal to this one, after successfully enrolling a card, in the `onResult` callback, in the `EnrollmentResult` object.

#### bannerImage: Bitmap?

Default value: `null`.

Will display the banner image that you set in this parameter at the top of the card details screen.

The banner image will take the device's width, but it has a fixed height of 100 dp.
The image view has the `android:scaleType="centerCrop"` attribute, which means that the banner image that you set will fill its entire predefined area.

For the banner image that you can set, we suggest to use the aspect ratio of the smallest devices that you support. On wider devices, the banner image will be cropped from top and bottom sides. This is because of the `android:scaleType="centerCrop"` attribute that we set for the image view.

If a device that opens the SDK has 320dp in width, the aspect ratio of the image view would be *320 : 100*.
If a device that opens the SDK has 475dp in width, the aspect ratio of your banner image would be *475 : 100*.

You need to provide the image for all screen densities.

Depending on what you want to display in the banner image, you might need to experiment a bit to make sure that nothing important from the image is hidden. The most important information should be displayed in the centre of the banner image.

#### programName: String

Default value: `"our"`

This value is used in the consent text when enrolling a card issued in a United States or Canada.

#### programType: ProgramType

Default value: `ProgramType.TRANSACTION_SELECT`

It specifies the type of program you want to enroll cards into. The `ProgramType` influences the flow that the SDK will show to cardholders when enrolling cards. 

> Note: For your Loyalty application, you need to use a Transaction Select program, so you can either explicitly set the value to `TRANSACTION_SELECT` or not set this property (because its default value is `TRANSACTION_SELECT`).

#### thirdPartyVerificationChoice: Boolean

Default value: `false`.

_Not useful for Loyalty/Select Transactions use cases, at the moment._

## Methods

### start(context: Context)

Starts a card enrollment flow. If you set the `programType` to:
1. `TRANSACTION_STREAM`, a card enrollment flow will be started, for a Transaction Stream program (usually used by Stream applications).
2. `TRANSACTION_SELECT`, a regular card enrollment flow will be started, for your Transaction Select program (usually used by Loyalty applications).

#### Parameters

- `context: Context`: the `Context` that will start the card enrollment flow. This is a *mandatory* parameter.

### verifyCard(context: Context, cardVerificationConfiguration: CardVerificationConfiguration)

_Not useful for Loyalty/Select Transactions use cases, at the moment._

### onMainActivityCreate(context: Context)

_Not useful for Loyalty/Select Transactions use cases, at the moment._

## Callbacks

The SDK provides the following callbacks:

### onResult: OnResultObserver

The `OnResultObserver` interface has a single function.

```kotlin
fun onResultAvailable(result: FidelResult)
```

It will be called when a `FidelResult` is available, during the card enrollment process. It can be called multiple times with different types of results.

#### sealed class FidelResult

Types of results:

- `Enrollment(val enrollmentResult: EnrollmentResult)`: Received after successfully enrolling the card in your Fidel API program. Please find details about `EnrollmentResult` below.
- `Verification(val verificationResult: VerificationResult)`._Not useful for Loyalty/Select Transactions use cases, at the moment._
- `Error(val error: FidelError)`. An error occurred either during the card enrollment process. Please find details about `FidelError` below.

#### data class EnrollmentResult

A result that can be received via the `onResult` callback, after a card is successfully enrolled in your Fidel API program.

Properties:
- `cardId: String`: The identifier of the card enrolled with your Fidel API program.
- `accountId: String`: The Fidel API account identifier.
- `programId: String`: The identifier of the program that the card was enrolled into.
- `enrollmentDate: Long`: The date and time when the card was enrolled, expressed as a `Long` value, that you can use with your preferred date & time APIs.
- `scheme: CardScheme`: The enrolled card's scheme.
- `isLive: Boolean`: This property will be `true` when your Fidel API account is live and the card was enrolled in your `live` Fidel API program. If the program that you enrolled the card into is not a `live` one, then this property will be `false`.
- `cardFirstNumbers: String?`: If available, this property will be populated with the first 6 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardLastNumbers: String?`: If available, this property will be populated with the last 4 numbers of the enrolled card. To turn on or off receiving these numbers, please check your Fidel API account's settings.
- `cardExpirationYear: Int`: The expiration year of the enrolled card. The values are four digit year values (ex: 2031), **not** shortened, two digit values (ex: 31).
- `cardExpirationMonth: Int`: The expiration month of the enrolled card. The values start with `1` (January) and end with `12` (December).
- `cardIssuingCountry: Country`: The country where the enrolled card was issued.
- `metaData: JSONObject?`: Custom data assigned to the enrolled card via the `metaData` SDK property.

#### data class VerificationResult

_Not useful for Loyalty/Select Transactions use cases, at the moment._

#### data class FidelError

A FidelError can occur during the card enrollment process. You can handle them via the `onResult` callback.

Properties:

- `message: String`: An error message explaining more details about the error. It is not localized.
- `date: Long`: The date and time when the error occurred, expressed as a `Long` value. You can use it with your preferred date & time APIs.
- `type: FidelErrorType`: The type of the error. Please check more details about `FidelErrorType` below.

#### sealed class FidelErrorType

Types of errors:

- `SdkConfigurationError`: The SDK [properties](#properties) configuration is incorrect or incomplete. You can receive this error as soon as you attempt to start a flow using the SDK [methods](#methods).
- `UserCanceled`: The user canceled the card enrollment flow at any stage.
- `DeviceNotSecure`: The device that the SDK is running on is not secure (for example, when it is rooted).
- `EnrollmentError(val type: EnrollmentErrorType)`. An error that is received during card enrollment or during consent creation. Please check more details about `EnrollmentErrorType` below.
- `VerificationError(val type: VerificationErrorType)`. _Not useful for Loyalty/Select Transactions use cases, at the moment._

#### enum class EnrollmentErrorType

An enum containing some of the cases of card enrollment errors.

Cases:

- `CARD_ALREADY_EXISTS`: The card was already enrolled in your Fidel API program. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `INVALID_PROGRAM_ID`: The program ID used to configure the SDK is not valid. If you receive this error, please make sure that you set a valid program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `map.already.exists`.
- `INVALID_SDK_KEY`: The SDK Key used to configure the Fidel SDK is not valid. If you receive this error, please make sure that you set a valid SDK Key via the `sdkKey` property. This error is equivalent to the Fidel API error with the code `credential`.
- `INEXISTENT_PROGRAM`: The program ID used to configure the Fidel SDK is of a program that does not exist. If you receive this error, please make sure that you set the correct program ID via the `programId` property. This error is equivalent to the Fidel API error with the code `item-exists`.
- `UNAUTHORIZED`: The card enrollment process is not authorized. This error is equivalent to the Fidel API error with the code `Unauthorized`.
- `UNEXPECTED`: An unexpected error during the card enrollment step.

The following list of cases are _not used_ by Loyalty/Select Transactions use cases, at the moment, so you can ignore them:

- `CARD_CONSENT_ISSUER_PROCESSING_CHARGE_ERROR`
- `CARD_CONSENT_DUPLICATE_TRANSACTION_ERROR`
- `CARD_CONSENT_INSUFFICIENT_FUNDS_ERROR`
- `CARD_CONSENT_PROCESSING_CHARGE_ERROR`
- `CARD_CONSENT_INCORRECT_CARD_DETAILS_ERROR`
- `CARD_CONSENT_CARD_LIMIT_EXCEEDED`
- `CARD_CONSENT_ERROR_GENERIC`


### onCardVerificationStarted: OnCardVerificationStartedObserver

_Not useful for Loyalty/Select Transactions use cases, at the moment._

### onCardVerificationChoiceSelected: OnCardVerificationChoiceSelectedObserver _(!Experimental)_

_Not useful for Loyalty/Select Transactions use cases, at the moment._
