---
title: Joys of accessing a bundle of a static framework in Xcode
date: "2024-08-04"
description: "A deep dive into a bug of a static framework resource bundle accessor."
---

As maintainers of Tuist, we get to work with so many interesting projects. Today was one of those days – this time I got to deep dive into the world of static framework symbols, resource bundle accessors, linking, and more.

The following exploration was sparked by [this issue](https://github.com/tuist/tuist/issues/6541) – GoogleMaps integration fails when using the Tuist [XcodeProj-based integration](https://docs.tuist.io/guides/develop/projects/dependencies#xcodeproj-codified-graphs).

### Understanding the problem

Whenever I hit an issue in Tuist, creating the smallest reproducible example helps a _ton_. You can find the reproducer [here](https://github.com/fortmarek/tuist-app-with-framework-with-static-framework-and-bundle-dependencies). What the graph in this scenario boils down to is:

```
App -> DynamicFramework -> StaticFrameworkWithResources
```

As you may know, static frameworks _can't_ host resources. What both SPM and Tuist do instead is to generate a new `.bundle` target that the `.staticFramework` will depend on. The new graph looks like this:

```
App -> DynamicFramework -> StaticFramework -> StaticFramework_StaticFramework (bundle)
```

Now let's try to run the app ... and what we end up with is:

```swift
StaticFramework/TuistBundle+StaticFramework.swift:37: Fatal error: unable to find bundle named StaticFramework_StaticFramework
```

That's unexpected! What are we doing wrong here? Let's dig into the `TuistBundle+StaticFramework.swift` accessor – for the record, SPM generates a very similar file at build time (whereas Tuist generates one during `tuist generate`):

```swift
private class BundleFinder {}
extension Foundation.Bundle {
/// Since StaticFramework is a static framework, the bundle containing the resources is copied into the final product.
static let module: Bundle = {
    let bundleName = "StaticFramework_StaticFramework"
    var candidates = [
        Bundle.main.resourceURL,
        Bundle(for: BundleFinder.self).resourceURL,
        Bundle.main.bundleURL,
    ]
    ...
    for candidate in candidates {
        let bundlePath = candidate?.appendingPathComponent(bundleName + ".bundle")
        if let bundle = bundlePath.flatMap(Bundle.init(url:)) {
            return bundle
        }
    }
    fatalError("unable to find bundle named StaticFramework_StaticFramework")
}()
}
```

The crash that we got corresponds to the `fatalError` at the bottom. So, how does the accessor bundle work and why does none of the candidates actually contain the bundle? The important bit here is the `BundleFinder` class at the top. This class serves as a way to locate the bundle based on where the static framework symbols are copied to.

In Tuist, the `StaticFramework_StaticFramework` bundle is copied to the first downstream target that can host resources. In our example, it's the `DynamicFramework`. In fact, if we take a look at the `Copy Bundle Resources` build phase of the `DynamicFramework`, we can find the `StaticFramework_StaticFramework.bundle` reference.

To double check, we can even go deeper and find the location of the bundle in the derived data. Based on the build phase, we'd expect `StaticFramework_StaticFramework.bundle` to be in `path-to-derived-data/Build/Products/Debug-iphonesimulator/App.app/Frameworks/DynamicFramework.framework/StaticFramework_StaticFramework.bundle`. And sure enough, there's the bundle! So what happens here?

Well, as mentioned above the `BundleFinder` finds the bundle based on where the static framework symbols are copied to. We can use a nifty command called [nm](https://www.man7.org/linux/man-pages/man1/nm.1.html) and search for `BundleFinder` in the `DynamicFramework.framework/DynamicFramework` executable:

```sh
nm path-to-derived-data/Build/Products/Debug-iphonesimulator/App.app/Frameworks/DynamicFramework.framework/DynamicFramework | grep BundleFinder
```

But this command doesn't find the `BundleFinder` symbol in the `DynamicFramework`. Where is it then? The only other executable in our example is `App`. Let's search for `BundleFinder` there instead:

```sh
nm path-to-derived-data/Build/Products/Debug-iphonesimulator/App.app/App | grep BundleFinder
```

This will indeed return some references:

```
000000010000582c t _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCADycfC
000000010000afac s _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCADycfCTq
0000000100005864 t _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCADycfc
000000010000b250 s _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMF
000000010000af6c s _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMXX
0000000100006ce4 t _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMa
0000000100011228 d _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMf
0000000100011200 d _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMm
000000010000af78 s _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCMn
0000000100011240 d _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCN
00000001000057f0 t _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCfD
00000001000057cc t _$s15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLCfd
0000000100010b28 s __DATA__TtC15StaticFrameworkP33_E84FB199CA5BF790D4BABEA53F2A398A12BundleFinder
0000000100010ae0 s __METACLASS_DATA__TtC15StaticFrameworkP33_E84FB199CA5BF790D4BABEA53F2A398A12BundleFinder
000000010000b10c s _symbolic _____ 15StaticFramework12BundleFinder33_E84FB199CA5BF790D4BABEA53F2A398ALLC
```

Why is this happening? Let's take a look at the `DynamicFramework` and `App` source files.

The only file of the `DynamicFramework` has the following content:

```
// DynamicFramework.swift
import Foundation
import StaticFramework

public enum DynamicFramework {
    public static func printFromDynamicFramework() {
        StaticFramework.printFromStaticFramework()

        print("print from DynamicFramework")
    }
}
```

And `App`'s `CustomView.swift` looks like this:

```
// AppDelegate.swift
import Foundation
import UIKit

public final class CustomView: UIView {
    private let label = UILabel()
    override public init(frame: CGRect) {
        super.init(frame: frame)

        label.font = UIFont(font: StaticFrameworkFontFamily.Poppins.regular, size: 20.0) // -> This is where we end up accessing the `BundleFinder` class!
        // ...
    }
}
```

As you can see, only `App` ends up using the `BundleFinder`. And the Swift compiler is smart enough here to know where `BundleFinder` is accessed and it only copies those symbols to where they are needed. This is actually useful, because otherwise the `DynamicFramework.framework` binary would get unnecessarily bloated with symbols that the framework doesn't need.

### Finding a solution

Ok, now we have a pretty good understanding of what's happening! But how do we fix the problem then? There are three solutions that I explored from here.

#### Force accessing the `BundleFinder` from the `DynamicFramework`

One, albeit hacky, way is to add the following line in `DynamicFramework.swift`:

```swift
_ = StaticFrameworkFontFamily.Poppins.regular
```

We're technically using the `BundleFinder` here and so the bundle accessor will now successfully find the `StaticFramework_StaticFramework.bundle` in the `DynamicFramework.framework`. Works, but yeah, hacky.

#### Set `GENERATE_MASTER_OBJECT_FILE` to `YES`

What the heck is `GENERATE_MASTER_OBJECT_FILE`? Let's look at the [documentation](https://developer.apple.com/documentation/xcode/build-settings-reference#Perform-Single-Object-Prelink) for this setting:

> Activating this setting will cause the object files built by a target to be prelinked using `ld -r` into a single object file, and that object file will then be linked into the final product. This is useful to force the linker to resolve symbols and link the object files into a single module before building a static library. (...)

In other words, we can override the default behavior of including static framework's symbols only where they are needed and instead copy _all_ the symbols into the first executable. In our case this would be the `DynamicFramework`. And indeed, it works!

If you are a Tuist user and run into this scenario, we recommend that you change this setting. Could we set this by default? Yes, but we could also unnecessarily bloat the final binary by including symbols that are not used.

SPM actually does set this [setting by default](https://github.com/tuist/tuist/issues/6320#issuecomment-2148580776), but at Tuist, we try to value explicitness [over convenience](https://docs.tuist.io/guides/develop/projects/cost-of-convenience#the-cost-of-convenience).

#### Change the `StaticFramework_StaticFramework.bundle` copy location

Alternatively, we can update the Tuist generation logic to copy the `StaticFramework_StaticFramework.bundle` directly to the `App` instead of `DynamicFramework`. Won't this mean that if `DynamicFramework` suddenly includes the `BundleFinder` symbols instead of `App` that we won't be able to find the bundle again? Yes! _If_ we depended solely on `BundleFinder`. But let's take a look at the resource bundle accessor candidates again:

```swift
    var candidates = [
        Bundle.main.resourceURL,
        Bundle(for: BundleFinder.self).resourceURL,
        Bundle.main.bundleURL,
    ]
```

While the second candidate would lead to a miss, the resource bundle accessor also looks into the `Bundle.main` – and that will be the `App`.

### What did we do in the end?

In the end, we decided to go with the last solution – but only for external (SPM) resources. While I would personally lean to setting the `GENERATE_MASTER_OBJECT_FILE`, this turned out not to be a viable solution for some SPM dependencies such as `GoogleMaps` that actually depend on the implicit SPM behavior and they expect the bundle to be _always_ in the main bundle.

For non-SPM resources, Tuist users can set the `GENERATE_MASTER_OBJECT_FILE` themselves if needed, although setting it by default when we detect it might be necessary is something we might consider in the future.

If you are interested in the final solution, here's the PR: https://github.com/tuist/tuist/pull/6565

Tschüss and see you at the next deep-dive 😛
