---
title: Sign in with Apple not supported for macOS apps distributed outside of Mac App Store
date: "2025-07-06"
---

**TL;DR:** Sign in with Apple is _not_ supported for macOS apps distributed outside of Mac App Store.

## How we ran into this

At Tuist, we have a [macOS](https://docs.tuist.dev/en/guides/features/previews#tuist-macos-app) app to make it dead simple to install [Tuist Previews](https://docs.tuist.dev/en/guides/features/previews#previews). We started to work on an [iOS app](https://community.tuist.dev/t/kicking-off-work-on-the-tuist-ios-app/570) to enable the same on iPhones (and possibly other devices from the Apple ecosystem). As part of this work, we're adding social logins – and to be able to get through the App Store review, we needed to add support for Sign in with Apple.

When adding support for Sign in with Apple, one needs to add an entitlement with the appropriate capability. And since the iOS and macOS codebase have common foundations, we implicitly added this capability to macOS. I haven't fully realized this as at the time since I was not adding an explicit Sign in with Apple button in macOS.

Things only started to fail when trying to release the macOS app:

> Provisioning profile "Tuist macOS Distribution" doesn't support the Sign in with Apple capability. (in target 'TuistApp' from project 'TuistApp')

## Finding the issue

When I first saw this, I thought I only had an outdated provisioning profile – and since we are distributing the macOS app oustide of the App Store, this provisioning profile is of type "Developer ID Application". So, I recreated one and double-checked it in the Apple Developer Account:

<img src="/img/sign-in-with-apple-not-supported-outside-mac-app-store/developer-account-screenshot.png" width=100% alt="Screenshot of the macOS Distribution provisioning profile, including Sign in with Apple capability"></img>

And, yes, the "Enabled capabilities" of provisioning profile explicitly include "Sign in with Apple". But re-releasing the app returned the same error!

So, I went to compare the iOS app provisioning profile (which worked) with the one for macOS – and surely enough, the downloaded macOS provisioning profile is _missing_ the Sign in with Apple capability:

<img src="/img/sign-in-with-apple-not-supported-outside-mac-app-store/provisioning-profile-macos.png" width=100% alt="Screenshot of the macOS Distribution provisioning profile in Finder without the Sign in with Apple capability"></img>

Whereas it is explicitly stated for the iOS profile:

<img src="/img/sign-in-with-apple-not-supported-outside-mac-app-store/provisioning-profile-ios.png" width=100% alt="Screenshot of the iOS Distribution provisioning profile in Finder with the Sign in with Apple capability"></img>

This spurred me to search more explicitly for Sign in with Apple in macOS apps distributed outside of Mac App Store and after a bit of searching, I landed on [this](https://developer.apple.com/forums/thread/671319?answerId=657361022#657361022) answer in the Apple Developer forum by the one-and-only Eskimo:

> Sign in with Apple is not supported for Developer ID. See [Developer Account Help > Reference > Supported capabilities (macOS)](https://help.apple.com/developer-account/#/devadf555df9) for a summary of which capabilities are available in which environments.

The Sign in with Apple capability missing in the Developer ID Application provisioning profile is the expected behavior. But what cost me the headache here was not the lack of support, but the fact that Apple happily lets you create a Developer ID Application provisioning profile with that capaibility, doesn't warn you about anything, and even _shows_ that capability in the Developer Account 🙃

Sometimes, developing for Apple platforms really is frustrating.

## "Fixing" the issue

To add support for Sign in with Apple in your macOS app distributed outside of Mac App Store, you have two "options":
- Move to Mac App Store
- Forward users to log in through your website (if you have it)

In the end, we've gone with the latter. We are a small team and going through the review cycle instead of continuously deploying our macOS app on every commit is not something we are willing to go through. Additionally, going to web to sign in doesn't feel like a huge UX regression, especially on Mac. Currently, we use the [https://developer.apple.com/documentation/authenticationservices/aswebauthenticationsession](ASWebAuthenticationSession) which spawns a Safari window to log in through our website, but you could open the browser directly and redirect with a deeplink URL (which is what `ASWebAuthentication` does as well – just for you).
