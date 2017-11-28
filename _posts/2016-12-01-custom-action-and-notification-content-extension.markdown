---
layout: post
title:  "Custom action and Notification Content Extension"
description: "Using `UNNotificationAction`, we can add button for user to response to the received notification. It can be attached to to any type of notification using category."
comments: true
date:   2016-12-01 13:46:52
keywords: "swift, uinotification, uitableviewcell, ios, UNNotificationAction"
category: swift
tags:
- swift
- ios
---
Beside adding image, GIF or video to notification, we can also allow user to interact with it. Using `UNNotificationAction`, we can add button for user to response to the received notification. It can be attached to to any type of notification using category.

## UNNotificationAction
Follow the setup below:

```swift
//1) Create action
let snoozeAction = UNNotificationAction(identifier: "Snooze", title: "5 more sec", options: [])
let stormOutAction = UNNotificationAction(identifier: "StormOut", title: "Cancel all Orders!", options: [])
let okayAction = UNNotificationAction(identifier: "Okay", title: "Okay", options: [.destructive])

//2) Attach to a category
let serveLaterCategory = UNNotificationCategory(identifier: "ServeLater", actions: [snoozeAction, okayAction],
  intentIdentifiers: [], options: [])
let cancelOrderCategory = UNNotificationCategory(identifier: CafeNotificationCategory.SoldOut.rawValue,
  actions: [stormOutAction, okayAction], intentIdentifiers: [], options: [])

//3 Register the category
UNUserNotificationCenter.current().setNotificationCategories([serveLaterCategory, cancelOrderCategory])
```

The codes at **1)** simply create three different `UNNotificationAction`. `identifier` is use to identify which action is tapped and `title` is the text displayed to the user.

**2)** group the different action into a `UNNotificationCategory`. Each will have its own unique `identifier` and come with a set of `actions`.

Then in **3)** we register the created categories with `UNUserNotificationCenter`.

In order to attach it to a notification, we will have to set the `categoryIdentifier` like this:

```swift
let content = UNMutableNotificationContent()
content.categoryIdentifier = "ServeLater"
// set title, text and sounds as well...
```
We would get a notification with action button like this:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/3/viewWithButton.png" width="220">

## Notification Content Extension
We could also go a step further and design our very own notification content view using `storyboard`.

In order to do that, we first must add a `Notification Content Extension`.

In the same project, Choose `File> New Target> Notification Content Extension`. Give it a name and click `Finish`, a new `Folder` with the given name will be added to your project:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/3/notificationFolder.png" width="500">

Some settings that we should be paying attention to in the `Info.plist` are:

- `UNNotificationExtensionCategory` - should match the `categoryIdentifier` when creating the `UNNotificationContent`.
- `UNNotificationExtensionDefaultContentHidden` - can toggle to show / hide the default `title` and `body` of the content.
- `UNNotificationExtensionInitialContentSizeRatio` - is the number we play around with to get the perfect size for the custom notification view.

`MainInterface.storyboard` contain a the view of the `NotificationViewController`. We could use it to set the design and layout of the content just like any view controller. However, direct interaction with the view is not possible thus `UIButton` would not work here.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/3/customContentStoryboard.png" width="600">

In `NotificationViewController`, we can use `protocol` of `UNNotificationContentExtension` to update the view with the received notification content.

We use a mandatory `didReceive` to update the text of our `@IBOulet UILabel`.

```swift
func didReceive(_ notification: UNNotification) {
    self.titleLabel.text = notification.request.content.title
    self.bodyLabel?.text = notification.request.content.body
}
```
If we need to handle the interaction on `UNNotificationAction`, we can conform to another protocol in `UNNotificationContentExtension`:

```swift
func didReceive(_ response: UNNotificationResponse, completionHandler completion: @escaping
  (UNNotificationContentExtensionResponseOption) -> Void) {
    if response.actionIdentifier == "StormOut" {
        UNUserNotificationCenter.current().removeAllPendingNotificationRequests()
        UNUserNotificationCenter.current().removeAllDeliveredNotifications()
    }
    completion(.dismiss)
}
```

Having implement all this, we would be able to get our very own custom notification view:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/3/customView.png" width="200">

## Sample Project
A completed project with different kind of notification setup could be access on [GitHub.](https://github.com/Zaccc123/cafe). Each `Order` button trigger a different kind of notification. Feel free to play around with it.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/3/newMenu.png" width="200">
