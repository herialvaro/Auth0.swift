# V2 MIGRATION GUIDE

Guide to migrating from `1.x` to `2.x`

## Supported platform versions

The deployment targets for each platform have been raised to:

- iOS 12.0
- macOS 10.15
- Mac Catalyst 13.0
- tvOS 12.0
- watchOS 6.2

## Supported languages

### Swift

The minimum supported Swift version is now 5.3.

### Objective-C

Auth0.swift no longer supports Objective-C.

## Supported JWT signature algorithms

ID Tokens signed with the HS256 algorithm are no longer allowed. 
This is because HS256 is a symmetric algorithm, which is not suitable for public clients like mobile apps.
The only algorithm supported now is RS256, an asymmetric algorithm.

If your app is using HS256, you'll need to switch it to RS256 in the dashboard or login will fail with an error:

**Your app's settings > Advanced settings > JSON Web Token (JWT) Signature Algorithm**

## Default values

### Scope

The default scope value in Web Auth and all the Authentication client methods (except `renew(withRefreshToken:scope:)`, in which `scope` keeps defaulting to `nil`) was changed from an assortment of values to `openid profile email`.

## Types removed

### Protocols

The following protocols have been removed:

- `AuthResumable`
- `AuthCancelable`
- `AuthProvider`
- `NativeAuthTransaction`

`AuthResumable` and `AuthCancelable` have been subsumed in `AuthTransaction`.

### Type aliases

The iOS-only type alias `A0URLOptionsKey` has been removed, as it is no longer needed.

### Enums

The custom `Result` enum has been removed, along with its shims. Auth0.swift is now using the Swift 5 `Result` type.

### Structs

The following structs have been removed, as they are no longer in use:

- `NativeAuthCredentials`
- `ConcatRequest`

### Classes

The following Objective-C compatibility wrappers have been removed:

- `_ObjectiveAuthenticationAPI`
- `_ObjectiveManagementAPI`
- `_ObjectiveOAuth2`

The following classes were also removed, as they are no longer in use:

- `Profile`
- `Identity`

## Metods Removed

The iOS-only method `resumeAuth(_:options:)` and the macOS-only method `resumeAuth(_:)` were removed from the library, as they are no longer needed.

### Authentication client

#### `login(usernameOrEmail:password:multifactorCode:connection:scope:parameters:)`

Use `login(usernameOrEmail:password:realm:audience:scope:)` instead.

#### `signUp(email:username:password:connection:userMetadata:scope:parameters:)`

Use `createUser(email:username:password:connection:userMetadata:rootAttributes:` and then `login(usernameOrEmail:password:realm:audience:scope:)` instead.

#### `tokenInfo(token:)` and `userInfo(token:)`

Use `userInfo(withAccessToken:)` instead.

#### `tokenExchange(withAppleAuthorizationCode:scope:audience:fullName:)`

Use `login(appleAuthorizationCode:fullName:profile:audience:scope:)` instead. 

The following methods have been removed and have no replacement, as they rely on deprecated endpoints:

- `loginSocial(token:connection:scope:parameters:)`
- `delegation(withParameters:)`

### Web Auth

Auth0.swift now only supports the [authorization code flow with PKCE](https://auth0.com/blog/oauth-2-best-practices-for-native-apps/), which is used by default. For this reason, the following methods have been removed from the Web Auth builder:

- `usingImplicitGrant()`
- `responseType(_:)`

The `useUniversalLink()` method was removed as well, as Universal Links [cannot be used](https://openradar.appspot.com/51091611) for OAuth redirections without user interaction since iOS 10.

### Credentials Manager

The method `enableTouchAuth(withTitle:cancelTitle:fallbackTitle:)` was removed. Use `enableBiometrics(withTitle:cancelTitle:fallbackTitle:evaluationPolicy:)` instead.

## Errors Removed

### `WebAuthError` enum

The following cases were removed, as they are no longer necessary:

- `noNonceProvided`
- `invalidIdTokenNonce`

## Types changed

- `UserInfo` was changed from class to struct

## Type properties changed

### `UserInfo` struct

It is now a struct, so its properties are no longer marked with the `@objc` attribute.

### `Credentials` class

The properties are no longer marked with the `@objc` attribute. Additionally, the following properties are no longer optional:

- `accessToken`
- `tokenType`
- `expiresIn`
- `idToken`

### `NSError` extension

These properties have been removed:

- `a0_isManagementError`
- `a0_isAuthenticationError`

## Method signatures changed

### Authentication client

#### Removed `parameters` parameter

The following methods lost the `parameters` parameter:

- `login(phoneNumber:code:audience:scope:)`
- `login(usernameOrEmail:password:realm:audience:scope:)`
- `loginDefaultDirectory(withUsername:password:audience:scope:)`
- `tokenExchange()`

To pass custom parameters to those (or any) method in the Authentication client, use the `parameters(_:)` method from `Request`:

```swift
Auth0
    .authentication()
    .tokenExchange() // Returns a Request
    .parameters(["key": "value"]) // 👈🏻
    .start { result in
        print(result)
    }
```

#### Reordered `scope` and `audience` parameters

In the following methods the `scope` and `audience` parameters switched places, for consistency with the rest of the methods in the Authentication client:

- `login(appleAuthorizationCode:fullName:profile:audience:scope:)`
- `login(facebookSessionAccessToken:profile:audience:scope:)`

#### Changed `scope` parameter to be non-optional

In the following methods the `scope` parameter became non-optional (with a default value of `openid profile email`):

- `login(email:code:audience:scope:)`
- `login(phoneNumber:code:audience:scope:)`
- `login(usernameOrEmail:password:realm:audience:scope:)`
- `loginDefaultDirectory(withUsername:password:audience:scope:)`
- `login(appleAuthorizationCode:fullName:profile:audience:scope:)`
- `login(facebookSessionAccessToken:profile:audience:scope:)`

#### Removed `channel` parameter

The `multifactorChallenge(mfaToken:types:authenticatorId:)` method lost its `channel` parameter, which is no longer necessary.

## Behavior changes

### `openid` scope enforced on Web Auth

If the scopes passed via the Web Auth method `.scope(_:)` do not include the `openid` scope, it will be added automatically.

```swift
Auth0
    .webAuth()
    .scope("profile email") // "openid profile email" will be used
    .start { result in
        print(result)
    }
```

### Credentials expiration on `CredentialsManager` 

The `CredentialsManager` class no longer takes into account the ID Token expiration to determine if the credentials are still valid. The only value being considered now is the Access Token expiration.

### Thread-safety when renewing credentials with the `CredentialsManager` 

The method `credentials(withScope:minTTL:parameters:callback:)` of the `CredentialsManager` class will now execute the credentials renewal serially, to prevent race conditions when Refresh Token Rotation is enabled.

## Title of change

Description of change

### Before

```swift
// Some code
```

### After

```swift
// Some code
```