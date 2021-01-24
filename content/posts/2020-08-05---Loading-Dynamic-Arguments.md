---
title: Loading Dynamic Arguments with the new ArgumentParser by Apple
date: "2020-08-05"
template: "post"
draft: false
slug: "loading-dynamic-arguments-argument-parser"
category: "CLI"
tags:
  - "CLI"
  - "Argument Parser"
  - "Swift"
description: "Apple has recently announced a new ArgumentParser library that leverages property wrappers and is a great example of a well-written declarative API. But the declarative nature of it comes with some drawbacks – mainly that if you need to do something custom that the library is not built for, you will need to get creative."
socialImage: "/media/argument_parser.jpg"
---

Apple has recently announced a new ArgumentParser library that leverages property wrappers and is a great example of a well-written declarative API. But the declarative nature of it comes with some drawbacks – mainly that if you need to do something custom that the library is not built for, you will need to get creative.

## Introduction
I have recently finished the migration from a parser that is used in TSCUtility library (currently still used by SPM) to the new one for tuist that helps with the maintenance and work with Xcode projects. Most of the migration was smooth 🙇‍♂️ with the exception of one command. That command is called `scaffold` and let me just quickly introduce it to you. When you run `tuist scaffold framework --name FrameworkName` it generates a new component in your project from a template called `framework`. Every template has its own manifest called `Template.swift` and has its own set of arguments.

What we want to do is parse `tuist scaffold framework`, the process that the user is creating a template called `framework`, then parse the manifest in `framework` directory and then add its arguments and let the ArgumentParser do the rest for us. Oof, this is a lot, we’ll go through this step-by-step, so don’t worry. 


## Starting out
Let’s get our hands dirty with some code 😉 If you want to follow along, you can download the starter project here. If you don’t, that’s fine, too 👍
This is what our command implementation initially looks like:

```swift
import ArgumentParser
import Foundation

struct Scaffold: ParsableCommand {
    @Argument()
    var template: String
    
    func run() throws {
        
    }
    
    static func preprocess(_ arguments: [String]) throws {
        // Obtaining template name
        let templateName = arguments[1]
        // Based on template name find its manifest
        let manifestPath = FileManager.default.currentDirectoryPath + "/\(templateName)/manifest.json"
        // Obtain data
        let data = try Data(contentsOf: URL(fileURLWithPath: manifestPath))
        // Parse the attributes
        let attributes: [String] = try JSONDecoder().decode([String].self, from: data)
        print(attributes)
    }
}
```

We are leveraging a custom function `preprocess` as we want to add the custom arguments before the parsing process starts. This is run before `Scaffold.main()` with `try? Scaffold.preprocess(CommandLine.arguments)` in `main.swift`. To properly handle errors you will need to define a custom `main` function for `ScaffoldCommand`, but that is out of scope for this tutorial. `Preprocess` function now finds `manifest.json` with the directory from the user input and parses the attributes defined there. In the example project we have an array of `[“name”]`. 

Let’s now try to run `scaffold framework --name FrameworkName`. But if you do so, you’ll get the following error: `Error: Unexpected argument 'FrameworkName'`. If you think about it, it makes sense – we are not defining a `--name` parameter, therefore the `ArgumentParser` has no chance to successfully parse the input.
Note About how ArgumentParser works
This is where we need to make a little detour to understand how ArgumentParser works under the hood in order to be able to inject our dynamic arguments. What we could boil our command down to is something like this:

```swift
struct Scaffold: ParsableCommand {
    @Argument()
    var template: String
}
```

As you can see, ArgumentParser is somehow magically able to recognize the arguments you want parsed just from the fact that you declare them with an appropriate property wrapper (in our case `@Argument`). If you want a full explanation of everything that happens during the parsing process, I’d recommend this great post. But what’s sufficient for us to know is that for every command ArgumentParser initializes `ArgumentSet`, which is what’s used in the subsequent parsing. If you look into the source code, this is how the initialization looks like:

```swift
extension ArgumentSet {
  init(_ type: ParsableArguments.Type) {
    let a: [ArgumentSet] = Mirror(reflecting: type.init())
      .children
      .compactMap { child in
        guard
          var codingKey = child.label,
          let parsed = child.value as? ArgumentSetProvider
          else { return nil }
        
        // Property wrappers have underscore-prefixed names
        codingKey = String(codingKey.first == "_" ? codingKey.dropFirst(1) : codingKey.dropFirst(0))
        
        let key = InputKey(rawValue: codingKey)
        return parsed.argumentSet(for: key)
    }
    self.init(additive: a)
  }
}
```

This is the line that interests us the most: `let a: [ArgumentSet] = Mirror(reflecting: type.init())`. In other words, ArgumentParser iterates through the children of the command’s mirror and that is how it’s able to magically recognize the arguments just from their declaration 🤯

## Leveraging our New Knowledge
With our new findings we should be able to inject our dynamic arguments. To do so we can use `CustomReflectable` where we will pass our custom array of children. But before doing that we need to save the attributes from `manifest.json` – however, this is pretty easy:

```swift
static var attributes: [String] = []

static func preprocess(_ arguments: [String]) throws {
    ...
    let attributes: [String] = try JSONDecoder().decode([String].self, from: data)
    Scaffold.attributes = attributes
}
```

We are just saving `attributes` to its `static` counterpart. We want it to be `static` because we have no instance of `Scaffold` command that we could use. Alas we can now add them to the `CustomReflectable` protocol implementation: 

```swift
extension Scaffold: CustomReflectable {
    var customMirror: Mirror {
        // #1
        let attributesChildren: [Mirror.Child] = Scaffold.attributes
            // #2
            .map {
                (name: $0, option: Option<String>(name: .shortAndLong))
            }
            // #3
            .map {
                Mirror.Child(label: $0.name, value: $0.option)
            }
        // #4
        let children = [
            Mirror.Child(label: "template", value: _template),
        ]
        // #5
        return Mirror(Scaffold(), children: children + attributesChildren)
    }
}
```

Soo, that’s a little bit less straightforward code, but let’s go through it:
In `#1` we are just iterating through the `attributes` array that we have declared. In `#2` we are initializing a tuple `(name: String, option: Option<String>)`. `name` is the name of the attribute, but what is `Option<String>`? Well, this is the property wrapper that you would normally declare this way:
```swift
@Option(name: .shortAndLong)
var name: String
```
But we obviously cannot do that, thus we need to initialize it directly. In `#3` we are then creating the `Mirror.Child` which is what we return in the end. In `#4` we are adding our `@Argument template`. For the value of the child we need to pass `_template` which is how we get the property itself, not its wrapped value (aka `Argument<String>`). And finally in `#5` we are just returning the `Mirror` itself with our initialized command and children of `attributes` and `template`.
Hopefully, we will now be able to simply run `scaffold framework --name FrameworkName` and all will go well. But I have bad news – it won’t 😞 If you do, this is the error you will receive:
"Argument `name` is defined without a corresponding `CodingKey`."
`CodingKey`, well, that sounds like somewhere decoding is failing 🤔

## Decoding
If you look at `ParsableCommand` protocol definition, you will see that it also conforms to `ParsableArgument`, which then conforms to `Decodable`. But what exactly is wrong? Well, in our custom mirror we told the `ArgumentParser` what arguments it should *expect*, but it does not parse the values themselves. If we look at our example, we successfully parse our `@Argument template`, but fail to decode the dynamic `--name` option. That is because `ArgumentParser` expects the `--name` option to be there, but when it gets to decoding it, it does not know how, since it is not defined in the compiler-generated `Decodable` implementation. But we can fix that 💪 Let’s just define our custom `init(from decoder: Decoder) throws` method – firstly, without our dynamic arguments:

```swift
enum CodingKeys: String, CodingKey {
    case template
}

init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    template = try container.decode(Argument<String>.self, forKey: .template).wrappedValue
}

// Necessary for conforming `ParsableArguments`
init() {}
```

Most of the code might be familiar to you – we define custom `CodingKeys` `enum` that contains our `template` argument and then we decode it as `String`. Simple! But we are still not handling our custom arguments ☝️

We’ll add another case to our `CodingKeys` `enum` called `dynamic(String)`. This will encapsulate all our dynamic arguments. Unfortunately, Swift is now not able to automatically convert the individual cases to `String`. We’ll give it a helping hand:

```swift
enum CodingKeys: CodingKey {
    case template
    case dynamic(String)
    
    init?(stringValue: String) {
        switch stringValue {
        case "template":
            self = .template
        case stringValue where Scaffold.attributes.contains(stringValue):
            self = .dynamic(stringValue)
        default:
            return nil
        }
    }
    
    var stringValue: String {
        switch self {
        case .template:
            return "template"
        case let .dynamic(name):
            return name
        }
    }

    // Not used
    var intValue: Int? { nil }
    init?(intValue _: Int) { nil }
}
```

We only need the computed `stringValue`and custom initializer from `stringValue`. 
And now the final piece of the puzzle:

```swift
// #1
var attributes: [String: String] = [:]

init(from decoder: Decoder) throws {
    let container = try decoder.container(keyedBy: CodingKeys.self)
    template = try container.decode(String.self, forKey: .template)
    // #2
    try Scaffold.attributes.forEach { name in
        attributes[name] = try container.decode(String.self, forKey: .dynamic(name))
    }
}
```
In `#1` we define a new property `attributes` that is a dictionary of `[String: String]` where the key will be the name of the attribute and the value will be … well, the value from the user input. In `#2` we iterate through `Scaffold.attributes` (not to be confused with our new property) and decoding them in a similar fashion as we did with `template` – now we are just saving our result to our new dictionary. 

## Finish line
Yep, we are nearing the finish line 🏁  Let’s make a final addition to our code and add the following to the `run` function:

```swift
func run() throws {
    print(template)
    print(attributes)
}
```

Now when you run `scaffold framework --name MyFramework` you will see:
```
framework
["name": "FrameworkName"]
```
And that is exactly what we want, finally 🥳  Now, we could even add a new `manifest.json` with `["platform"]` to the `app` directory. When we call `scaffold app --platform iOS` everything still works! 
Recap of what we have just achieved:
1. Preprocess our input and recognize the name of template
2. Load a `.json` file in a directory of the preprocessed template
3. Parse `.json` and dynamically add its attributes to `ParsableCommand`
4. Created a Swift CLI with ArgumentParser that can now work with dynamic attributes ✅
I feel that’s a lot 😉 You can go ahead and play with the final project which you can find here. 


*Originally published at [Ackee](https://www.ackee.cz/blog/en/argumentparser-loading-dynamic-arguments/).*