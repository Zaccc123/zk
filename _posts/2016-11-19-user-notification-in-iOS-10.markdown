---
layout: post
title:  "User Notifications in iOS 10!"
description: "With iOS 10, beside just text base notification, we can now add other items like images, GIF or even video clips. In this post, we will create a simple text notification, followed by image and GIF."
date: 2016-11-19 13:46:52
comments: true
keywords: "swift, uinotification, ios, dynamic, layout"
category: swift
tags:
- swift
- ios
---
With iOS 10, beside just text base notification, we can now add other items like images, GIF or even video clips. In this post, we will create a simple text notification, followed by image and GIF.


## Setup
The minimum setup to create a local notification require four thing:

- Authorization (one time)
- Content
- Trigger
- Request

### Authorization
Before creating any notification, we need to ask user for permission to receive notification. Add the following in `AppDelegate`:

```swift
let center = UNUserNotificationCenter.current()
center.requestAuthorization (options: [.alert, .sound]) {(accepted, error) in
    if !accepted {
        print("User denied authorisation.")
    }
}
```
We could also ask for more `options` like `.badge` if needed.


### Notification Content
An instance of `UNMutableNotificationContent` is required. We should at least be  setting the notification content with `title`, `body` and `sound`.

```swift
func createNotificationContent(title: String, text: String) -> UNNotificationContent {
    let content = UNMutableNotificationContent()
    content.title = title
    content.body = text
    content.sound = UNNotificationSound.default()

    return content
}
```
We could set more properties if needed. A list of properties could be found in the docs [here](https://developer.apple.com/reference/usernotifications/unmutablenotificationcontent).

### Trigger
In order to trigger a notification, we could use either of the following three option `UNTimeIntervalNotificationTrigger`, `UNCalendarNotificationTrigger` or `UNLocationNotificationTrigger`.

**`UNTimeIntervalNotificationTrigger`** is the simplest of all. We just pass in a `timeInterval` for seconds later to trigger the notification.

```swift
let trigger = UNTimeIntervalNotificationTrigger(timeInterval: 5, repeats: false)
```

**`UNCalendarNotificationTrigger `** require a `Calendar` A example to post a notification for the next day could be as follow:

```swift
let date = Date(timeIntervalSinceNow: 86400)
let tomorrow = Calendar.current.dateComponents([.year,.month,.day,.hour,.minute,.second,], from: date)
let trigger = UNCalendarNotificationTrigger(dateMatching: tomorrow, repeats: false)
```

**`UNLocationNotificationTrigger`** trigger a notification when user enters or leave a `CLRegion`.

```swift
let trigger = UNLocationNotificationTrigger(triggerWithRegion: region, repeats:false)
```

### Request
Having both the content and trigger ready, we could easily create a request now.

```swift
let request = UNNotificationRequest(identifier: "UniqueName", content: content, trigger: trigger)
UNUserNotificationCenter.current().add(request) { error in
    if let error = error {
        print(error)
    }
}
```
`identifier` is a unique string given to each request. If the same `identifier` is passed, the existing notification will be replaced with the latest one.

## Sample Project
I have created a simple cafe ordering app. You will able to order three item, each showing a different type of local notification after a short interval.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/2/menu.png" width="220">
<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/2/notification_image.png" width="220">
<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/2/notification_force_touch.png" width="220">

The project can be clone from [GitHub.](https://github.com/Zaccc123/cafe) After clicking on a `order`, user should leave the app to wait for notification. Local notification will not be display when the app is on foreground.

The project also uses `UNNotificationAction` that I will write about in the next post.
