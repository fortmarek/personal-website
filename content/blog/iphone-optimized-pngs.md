---
title: iPhone-optimized PNGs
date: "2025-06-29"
description: "How PNGs in .ipa archives are not PNGs at all"
---

As part of [Tuist Previews](https://docs.tuist.dev/en/guides/features/previews#previews), we upload app icons that are present in the `.ipa` archive, so we can present them in our dashboard or in our [macOS app](https://docs.tuist.dev/en/guides/features/previews#tuist-macos-app).

However, when we uploaded those icons, they would fail to show in browsers other than Safari. That was very fishy 🐟

## Getting the app icon from `.ipa`

`.ipa` archives are in effect simple `.zip` archives that have a conventional custom extension name. The content of the unarchived `.ipa` is typically `Payload/YourApp.app` where `YourApp.app` is the actual app bundle that you end up installing on your device. Inside the `.app` bundle, you'll find an `Info.plist`.


The `Info.plist` contains, among other things, the name of the icon file specified under `CFBundleIcons` > `CFBundlePrimaryIcons` > `CFBundleIconFiles`. In our case, the value of `CFBundleIconFiles` was a single item called `AppIcon60x60`. And surely enough, the `.app` bundle contained a file called `AppIcon60x60@2x.png`:

<img src="/img/iphone-optimized-pngs/contents-of-app-bundle.png" width=500px alt="Screenshot of Finder with contents of the .app bundle"></img>

Perfect! We can take this icon file and upload it to our storage since it's a PNG image ... right?

## PNGs with iPhone optimizations

When `xcodebuild` archives the app, it "optimizes" the icons for iPhone (and other platforms) and converts the PNG to a [PNG CgBI format](https://theapplewiki.com/wiki/PNG_CgBI_Format):
> To optimize for the native pixel format of the iPhone's early PowerVR GPUs, Apple implemented a non-standard PNG format where the red and blue pixels are flipped (BGRA instead of RGBA). The format additionally includes extra data before the PNG header, and compressed image data without the traditional headers and footers. All iPhone PNG images appear to follow this format.

In other words, when you upload a PNG contained in the `.ipa` archive, those PNGs are actually not PNGs at all – they are custom image files masquerading as PNGs. And that's also why apps made by Apple, like Preview or Safari, _can_ show these images, but other apps/browsers usually don't have support for these (because, why would they have support for odd Apple formats).

## Converting back to a standard PNG format

Fortunately, Apple at least did consider that there might be good use cases when folks would like an actual PNG over the "iPhone-optimized" image. There's a tool called [pngcrush](https://pmt.sourceforge.io/pngcrush/) that actually _ships with Xcode_ and has a command for reverting iPhone optimziations:

```sh
xcrun pngcrush -revert-iphone-optimizations iphone-optimized.png standard.png
```

And just like that, we actually get back a _normal_ PNG image 🎉 And that's exactly how we fixed the issue in the Tuist CLI: [#7705](https://github.com/tuist/tuist/pull/7705)

At the end of the day, I'm sure at the time there were good reasons to do these PNG adjustments – but I do wish Apple made it obvious and came up with a custom extension instead of reusing the PNG extension for a file that's not a standard PNG and fails to load in most apps not owned by Apple.
