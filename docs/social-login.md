# Social Login

Module: `Zero_SocialLogin`. Customers sign in and sign up with Google,
Apple or Facebook. The buttons are on the sign-in page, the registration
page and the checkout. Customers connect or disconnect those accounts under
**My Account > Sign-In Methods**. Admin sign-in is not affected.

No JavaScript and no third-party libraries. The module implements OpenID
Connect (Google, Apple) and OAuth 2.0 (Facebook) itself. The buttons are
plain links, so the pages stay cacheable. The module changes no other
module's files: it adds its blocks through layout. Without it, the pages
look as before.

## Turning it on

**Stores > Configuration > Zero > Social Login**: set **Enabled**, then
switch on each service and enter its keys. A service shows its button only
when it is switched on and its keys are filled in.

Each service needs the store's **redirect URI**. The settings show it under
each service's **Enabled** field. It looks like this:

```
https://shop.example/sociallogin/login/callback/provider/google/
```

With store codes in URLs, each store view has its own redirect URI.
Register each one.

### Google

1. [Google Cloud Console](https://console.cloud.google.com/) > APIs &
   Services > **OAuth consent screen**: app name, support email, the
   store's domain. Scopes: `openid`, `email`, `profile` (no review needed).
   Publish the app ("In production"). In testing mode, only listed test
   users can sign in.
2. **Credentials > Create credentials > OAuth client ID**, type **Web
   application**. Under **Authorized redirect URIs**, add the redirect URI.
3. Copy the **Client ID** and **Client Secret** into the settings.

### Apple

Needs a paid Apple Developer account. Apple accepts HTTPS redirect URIs on a
real domain only, so it can't be tried on `localhost`.

1. [Apple Developer](https://developer.apple.com/account/resources/) >
   **Identifiers > App IDs**: an App ID with **Sign in with Apple** enabled
   (any name, e.g. `com.example.shop`).
2. **Identifiers > Services IDs**: a new Services ID (e.g.
   `com.example.shop.signin`). This is the **Services ID** in the settings.
   Enable Sign in with Apple, **Configure**: the App ID from step 1, the
   store's domain under **Domains and Subdomains**, the redirect URI under
   **Return URLs**.
3. **Keys**: a new key with Sign in with Apple, linked to the App ID.
   Download the `.p8` file (only once) and note the **Key ID**.
4. Settings: Services ID, **Team ID** (top right of the developer account),
   Key ID, and the `.p8` file's contents as **Private Key**.

Apple gives the customer's name only on their first sign-in. The email can
be a private relay address (`…@privaterelay.appleid.com`) that forwards to
the customer. To email those addresses, register the store's sending domain
under **Services > Sign in with Apple for Email Communication**.

### Facebook

1. [Meta for Developers](https://developers.facebook.com/apps/) > **Create
   app**, use case **Authenticate and request data from users with Facebook
   Login**.
2. Use case **Customize**: add the `email` permission (next to
   `public_profile`). Under **Settings**, add the redirect URI to **Valid
   OAuth Redirect URIs**.
3. **App settings > Basic**: privacy policy URL, the store's domain under
   App Domains. Copy the **App ID** and **App Secret** into the settings.
4. Switch the app to **Live**. In development mode, only the app's own
   users can sign in.

## How sign-in decides which account

1. The service account is already connected to a store account: that
   account signs in.
2. A store account has the same email:
   - **Google, Apple**: they state the email is verified. The service is
     connected to that account and the customer signs in.
   - **Facebook** doesn't say whether the email is verified, so it never
     takes over an existing account. The customer is asked to sign in with
     their password once and connect Facebook under My Account.
3. Otherwise a new account is created, without a password. Magento's
   welcome email goes out. The customer can set a password later with
   **Forgot Your Password?**.

**Require Emails Confirmation** (Customer Configuration) is respected. A
verified email (Google, Apple) counts as confirmed. Otherwise (Facebook)
Magento's confirmation link is sent, and the customer signs in after
opening it.

With customer accounts shared per website, a service account can connect
to one account per website.

A customer can't disconnect their only way in: an account without a
password keeps at least one service.

After signing in, the customer goes where Magento's sign-in redirect
settings send them (by default the account dashboard). From the checkout,
they go back to the checkout. The guest cart is merged as with a password
sign-in.

## Security

- **state**: a random value per attempt, kept in the session and checked on
  the way back. An attempt is used once and expires after 15 minutes. A
  sign-in link can't be replayed in another browser.
- **PKCE** (S256) with Google and Facebook. Apple doesn't support it.
- **nonce** in the ID token (Google, Apple), checked with the issuer,
  audience and expiry. The ID token comes straight from the service's token
  endpoint over TLS, which OpenID Connect accepts in place of a signature
  check (Core 1.0, §3.1.3.7).
- **appsecret_proof** on Facebook's Graph API calls.
- Secrets are stored encrypted (Magento's encrypted config fields).
- Apple returns with a cross-site POST, which carries no session cookie.
  Magento doesn't start a session for that request (as Magento_Paypal does
  for its returns), so the visitor's own session and cart stay as they
  are. The POST is turned into a GET, and the attempt is checked there.

## For developers

| | |
| --- | --- |
| `zero_social_login_identity` | Connected accounts: `customer_id`, `provider`, `subject` (the service's user ID), `email`, `created_at`, `last_login_at`. One per customer and service; deleted with the customer |
| `sociallogin/login/start` | A button: opens an attempt and redirects to the service |
| `sociallogin/login/callback` | The redirect URI |
| `sociallogin/account` | My Account > Sign-In Methods; `…/disconnect` (POST) |

Another service: implement `Model\Provider\ProviderInterface`, add it to
`Model\Provider\Pool`'s `providers` in `di.xml`, and add a settings group
`zero_social_login/<code>` with `enabled`. Its logo goes in
`ViewModel\Buttons::ICONS`.

`Model\Authenticator` decides the account (see above). The checkout
buttons come from the module's `zero_checkout_index.xml`: it gives
Zero_Checkout's sign-in block this module's template, which renders
Zero_Checkout's own template first.

### Tests

`dev/tests/zero-e2e/tests/social-login.spec.ts`: the buttons, the request
sent to each service (state, nonce, PKCE, redirect URI), a forged or
foreign callback, cancelling, Apple's cross-site POST keeping the session
and cart, checkout, My Account. The services themselves can't be reached
from a test. Unit tests in `Test/Unit`: ID token checks, Apple's ES256
client secret, each service's requests and profile, and the account rules.

## Disabling or removing

Its data lives in its own tables (`zero_social_login_identity`). With the module's code still in
place, `bin/magento module:disable Zero_SocialLogin` followed by `setup:upgrade`
**drops those tables** (Magento's declarative schema), and enabling the module
again brings them back empty. To keep the data, switch the feature off in
Stores > Configuration where it has a setting, or back up first:
`bin/magento setup:upgrade --safe-mode=1` dumps the data before anything is
dropped (`--data-restore=1` puts it back). Deleting the module's code instead
leaves the tables in place.
