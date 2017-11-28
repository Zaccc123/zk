---
layout: post
title: "State Pattern to Manage Reuseable View Controller"
description: "Using State Pattern to write cleaner and reuseable view controller."
date: 2017-01-21 16:39:18
comments: true
keywords: "swift, state, pattern, view, controller, ios"
category: swift
tags:
- swift
- ios
---

Is quite common to have a `ViewController` being able to access by different type of `User`. `GuestUser`, `NormalUser`, `AdminUser` and etc. Very often the different in the `ViewController` is subtle. We could reuse most code in the `ViewController` but have to hide certain `UIControl`.

Most of the time, clients / bosses would come to us with this:

"Oh, `AdminUser` should be able to see and do everything."

"Oh, `LoggedInUser` should be able to see this and but not do that."

"Oh, `GuestUser` should be able to see this only."

### Common Solution
The most common solution is to use `enum` and `switch`. Something like this:

```swift
func toggleControls(userType: UserType) {
    switch userType {
    case .adminUser:
        navigationItem.rightBarButtonItem?.isEnabled = true
        infoView.isHidden = false
    case .loggedInUser:
        navigationItem.rightBarButtonItem?.isEnabled = true
        infoView.isHidden = true
    case .guestUser:
        navigationItem.rightBarButtonItem?.isEnabled = false
        infoView.isHidden = true
    }
}
```

This is okay if all we have to toggle are accessible on load. But if we have to modify the behaviour of a `cell` in a `tableView`, we can't do this anymore. Rather, we need to duplicate the `switch` code into either the `customCell` or in `cellForRowAtIndexPath`.

In short, duplication is bad. `switch` tend to get difficult to maintain when more `case` is added. Especially when we have to add in logic or `if else` in each `case`.

### Using State Pattern
Now, if we were to use State pattern, all we need is the following in the `ViewController`.

```swift
var state: State? {
    didSet {
        state?.canAdd(barButtonItem: navigationItem.rightBarButtonItem)
        state?.canSeeInfo(view: infoView)
    }
}

state?.canSelectCell(cell: cell) //in cellForRowAtIndexPath
```

So in order to achieve the above, we need to move the logic into a `State` `protocol`.

```swift
protocol State {    //1
    func canAdd(barButtonItem: UIBarButtonItem?)
    func canSeeInfo(view: UIView)
    func canSelectCell(cell: UITableViewCell)
}

struct AdminState: State {  //2
    func canAdd(barButtonItem: UIBarButtonItem?) {
        barButtonItem?.isEnabled = true
    }

    func canSeeInfo(view: UIView) {
        view.isHidden = false
    }

    func canSelectCell(cell: UITableViewCell) {
        cell.accessoryType = .disclosureIndicator
        cell.isUserInteractionEnabled = true
    }
}

struct UserState: State {   //3
    func canAdd(barButtonItem: UIBarButtonItem?) {
        barButtonItem?.isEnabled = false
    }

    func canSeeInfo(view: UIView) {
        view.isHidden = false
    }

    func canSelectCell(cell: UITableViewCell) {
        cell.accessoryType = .disclosureIndicator
        cell.isUserInteractionEnabled = true
    }
}

struct GuestState: State {  //4
    func canAdd(barButtonItem: UIBarButtonItem?) {
        barButtonItem?.isEnabled = false
    }

    func canSeeInfo(view: UIView) {
        view.isHidden = true
    }

    func canSelectCell(cell: UITableViewCell) {
        cell.accessoryType = .none
        cell.isUserInteractionEnabled = false
    }
}
```

This is the **machine** that is doing all the settings. No `if-else` or any `switch-case`.

**1)** uses `protocol` to define what is needed when another `class` or `struct` confront to this `protocol`.

**2) 3) 4)** are just `struct` that confront to `State` protocol. We will receive a error if we miss out on any `func` define in `State`.

On first look, it look like there is more lines of code, and yes is true. However, the **really good** things about this is:

1. We do not have to worry about adding new case in `ViewController`. Just create a new `struct` that confront to the `State` protocol.
2. We do not have to worry about forget to handle the state of certain item. Having multiple `switch-case` in `ViewController` and `UITableViewCell`, sometimes we forget to update certain item.
3. One look is all we need to know what is happening in each individual `struct`.

### Summary
When we have a perfect requirement gathering phrase and design is drafted out to handle different case, we tend to have a good separation of concern. This allow us to plan ahead and design `ViewController` that only do one thing.

But most of the time, this is not the case. Requirement change or get added spontaneously. If you notice that `ViewController` getting huge and different condition is added, try using the `State Pattern`.

For more information about `State Pattern`, it can be read up [here](https://github.com/ochococo/Design-Patterns-In-Swift#-state).
