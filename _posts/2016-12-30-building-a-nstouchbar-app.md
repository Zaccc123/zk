---
layout: post
title: "Building a NSTouchBar App"
description: "In this tutorial, I will be building an app that uses the NSTouchBar API."
date: 2016-12-30 16:41:50
comments: true
keywords: "swift, nstouchbar, touchbar, ios, touchbarapi, api"
category: swift
tags:
- swift
- ios
---

The new MacBook Pro with its fancy touch bar has been released by Apple. Being a developer, we always look to toy with the newest technology. In this tutorial, I will be building an app that uses the NSTouchBar API.

### Requirements
We do **not need** the latest MacBook Pro to develop this app. All you will need is to be at least on the following:

- macOS Sierra 10.12.1 (Build 16B2657)
- Xcode Version 8.1

To check which version are you on, do this: ` > About This Mac`. To see the `Build`, click on the version number `Version OS 10.12.1`.

The following tutorial is developed on `macOS Sierra 10.12.2 (16C67)` and `Xcode Version 8.2`.

## Overview
We will be building a simple application that ask: **"How do you feel today?"** User will be able to express their feeling by touching a emoji button on the touch bar. A short text will be shown in order to response to their feeling.

### Creating the project
From Xcode, select `File > New > Project`. Click on `macOS` tab and select `Cocoa Application`. Then enter all the required detail.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/5/createProject.png" width="600">

Before we start, we should tell Xcode that we would like to show the `Touch Bar`. Go to `Window > Show Touch Bar`. Upon clicking `Show Touch Bar`, a `Touch Bar` should now appear on the screen.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/5/showTouchBar.png" width="600">

### Set Up
Next, we need to allow the customisation of `Touch Bar` for our application. In `AppDelegate`, do this:

```swift
func applicationDidFinishLaunching(_ aNotification: Notification) {
    if #available(OSX 10.12.2, *) {
        NSApplication.shared().isAutomaticCustomizeTouchBarMenuItemEnabled = true
    }
}
```
By default, this value is set to `false`. In order to customise the `Touch Bar` we need to enable it. The line `if #available(OSX 10.12.2, *)` is a check as we are using the `Touch Bar` that does not exist in previous OSX version. Xcode will throw an error whenever we use something that is not available in previous version.

We will then create a `HYFTWindowController` a subclass of `NSWindowController`. Go to the `Main` storyboard and update the `Window Controller` class in inspector with our newly created `HYFTWindowController`.

### Start Coding Touch Bar
With all the setup completed, we are now ready to start writing code for the `Touch Bar`. In `HYFTWindowController`, do this:

```swift
@available(OSX 10.12.2, *)
override func makeTouchBar() -> NSTouchBar? {
    guard let viewController = contentViewController as? ViewController else {
        return nil
    }
    return viewController.makeTouchBar()
}
```

With `HYFTWindowController` being our `firstResponder`, the `makeTouchBar()` would get called first. Here, we would pass the job to our `ViewController` to create the `Touch Bar`.

In `ViewController`, `override` the `makeTouchBar()` like this:

```swift
@available(OSX 10.12.2, *)
override func makeTouchBar() -> NSTouchBar? {
    let touchBar = NSTouchBar()
    touchBar.delegate = self
    touchBar.customizationIdentifier = .hyftBar
    touchBar.defaultItemIdentifiers = [.infoLabelItem, .joyEmojiItem, .sadEmojiItem,
      .angryEmojiItem, .flexibleSpace, .otherItemsProxy]
    return touchBar
}
```

Here, we create a new `NSTouchBar` and set ourself as its `delegate`. The `touchBar.customizationIdentifier` required a unique string to identify each `touchBar`.

The `touchBar.defaultItemsIdentifiers` expect an array of unique string identifiers to help us setup the item later. Note that we also pass in the `.flexibleSpacing` and `.otherItemsProxy`. These are a set of `NSTouchBarItemIdentifer` provided by `AppKit`. These help us set flexible space between item and allow other `touch bar` with visible item to be nested inside.

A list of other default items can be found [here](https://developer.apple.com/reference/appkit/nstouchbaritemidentifier).

In order to better manage our own identifier, I created a new file `HYFTIdentifier` and have the following:

```swift
import AppKit

extension NSTouchBarItemIdentifier {
    static let infoLabelItem = NSTouchBarItemIdentifier("com.zeta.InfoLabel")
    static let joyEmojiItem = NSTouchBarItemIdentifier("com.zeta.JoyEmoji")
    static let sadEmojiItem = NSTouchBarItemIdentifier("com.zeta.SadEmoji")
    static let angryEmojiItem = NSTouchBarItemIdentifier("com.zeta.AngryEmoji")

}

extension NSTouchBarCustomizationIdentifier {
    static let hyftBar = NSTouchBarCustomizationIdentifier("com.zeta.ViewController.HYFTTouchBar")
}
```

### Creating NSTouchBarItem
Finally, we are all ready to create items on the touch bar. Is actually pretty straightforward to create items using the `touchBar(_ touchBar: NSTouchBar, makeItemForIdentifier identifier: ` method.

To make our code more readable, do this in `ViewController`:

```swift
@available(OSX 10.12.2, *)
extension ViewController: NSTouchBarDelegate {

    func touchBar(_ touchBar: NSTouchBar, makeItemForIdentifier identifier: NSTouchBarItemIdentifier) ->
      NSTouchBarItem? {
        let custom = NSCustomTouchBarItem(identifier: identifier)

        switch identifier {
        case NSTouchBarItemIdentifier.infoLabelItem:
            let label = NSTextField.init(labelWithString: NSLocalizedString("How do you feel today?", comment:""))
            custom.view = label

        case NSTouchBarItemIdentifier.joyEmojiItem:
            custom.view = NSButton(title: NSLocalizedString("😂", comment:""), target: self,
              action: #selector(buttonPressed(_:)))

        case NSTouchBarItemIdentifier.sadEmojiItem:
            custom.view = NSButton(title: NSLocalizedString("😟", comment:""), target: self,
              action: #selector(buttonPressed(_:)))

        case NSTouchBarItemIdentifier.angryEmojiItem:
            custom.view = NSButton(title: NSLocalizedString("😡", comment:""), target: self,
              action: #selector(buttonPressed(_:)))

        default:
            return nil
        }
        return custom
    }
}
```

The `identifier` would help us identify which item are we going to be creating. We use a `switch` here to setup each item differently. The order this get trigger will be the same as the order of item in the array passed into `touchBar.defaultItemIdentifiers`.

The `NSCustomTouchBarItem` have a `view` that we should use for setting up custom item. For `.infoLabelItem`, we just create a `NSTextField` and update it. For the rest, we create a `NSButtom` with different `title` all of each will trigger a action `buttonPressed(_:)`.

In order to prevent errors, we will create a empty `buttonPressed(_:)` in the `ViewController` and run the app for now. You should see this in your `Touch Bar`.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/5/touchBar.png" width="800">

### Completing the App
What is left now is to complete the app. In the `Main` storyboard, setup the `ViewController` like this:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/5/setupViewController.png" width="800">

One `NSTextField` on the top and another hidden `NSTextField` near the bottom. Then add three `NSButton` in the middle with the "😟", "😟", "😡" as the respective `title`.

Create a `@IBOulet messageTextField` and hook it up to the bottom `NSTextField`.Then hook up all three `NSButton` action to the `buttonPressed(_:)` method in `ViewController`.

Next, copy and paste the following into `ViewController`.

```swift
func getMessageBaseOnReaction(_ reaction: String) -> String {
        switch reaction {
        case "😂":
            return "Being happy never goes out of style. - Lilly Pulitzer"
        case "😟":
            return "Sadness flies away on the wings of time. - Jean de La Fontaine"
        case "😡":
            return "To be angry is to revenge the faults of others on ourselves. - Alexander Pope"
        default:
            return "Look like our quote does not cater to this reaction"
        }
    }

//Quotes are extracted from https://www.brainyquote.com
func showMessageBaseOnReaction(_ reaction: String) {
    messageTextField.isHidden = false
    messageTextField.stringValue = getMessageBaseOnReaction(reaction)
}

//MARK: IBActions
@IBAction func buttonPressed(_ sender: NSButton) {
    showMessageBaseOnReaction(sender.title)
}
```
The above are just code to update the `messageTextField ` with quote base on the action by the user.

Now, run the app again and play with the `Touch Bar`.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/5/final.png" width="800">

Clicking any emoji item on the `Touch Bar` or the `button` on the window will both trigger the text to change. Now we have our very own `Touch Bar` app 😂.

The completed project can be access via GitHub [here](https://github.com/Zaccc123/HYFTTouchBarApp).

### Where to go from here?
We have only touch on the very basic of `NSTouchBar`. There is a lot more customisation and control we could use to enhance user experience with the app. The following resources should be able to help you understand better:

- [Apple Sample NSTouchBar Catelog Project](https://developer.apple.com/library/content/samplecode/NSTouchBarCatalog/Introduction/Intro.html)
- [How to Use NSTouchBar on macOS by RayWenderlich.com](https://www.raywenderlich.com/147118/use-nstouchbar-macos)
- [NSTouchBar CheatSheet and more Swift Example](https://github.com/loretoparisi/touchbar)
