---
layout: post
title: "Handling API Response with enum"
description: "Using enum to help us handle API Response better!"
date: 2017-01-15 16:38:20
comments: true
keywords: "swift, api, response, ios, enum"
category: swift
tags:
- swift
- ios
---

Coming from `Objective-C` we tend to bring over what we use to do into `swift`. One thing that I am so use to doing are the way network response is handled, usually from API calls.

```swift
// DrinkService
func getDrinks(completion: ([Drink], Error?) -> Void) {
    //some api call to get drinks from our server
}

//ViewController
DrinkService().getDrinks() { drinks, error in
    if error == nil {
        self.drinks = drinks
        self.tableView.reloadData()
    } else if let errorMessage = error?.localizedDescription{
        showErrorAlertView(message: errorMessage)
    }
}
```

Many times, we return the required object and `Error` in the completion method. We often check the `Error` is not nil then we do the happy flow.

This is not clean because irregardless of the result the completion always return the same number of argument. We really only need one in most cases. Using `enum` with associated values, we can solve this with ease. It also make it more readable.

### Enum with associated values

```swift
enum Result {
    case success([Drink])
    case error(Error)
}
```

With the above, we can now change our following function into this:

```swift
// DrinkService
func getDrinks(completion: (Result) -> Void) {
    //some api call to get drinks from our server
}

func getDrink() {
    DrinkService().getDrinks() { result in
        switch result {
        case .success(let drinks):
            self.drinks = drinks
            self.tableView.reloadData()
        case .error(let error):
            showErrorAlertView(message: error.localizedDescription)
        }
    }
}
```

We can make it reusable for other service function by using `generic`. Change the enum into this:

```swift
enum Result<T> {
    case success(T)
    case error(Error)
}
```
We also have to update the `getDrinks` function into this:

```swift
func getDrinks(completion: (Result<[Drink]>) -> Void) {
    //some api call to get drinks from our server
}
```

The only change is to specified what is `T` in the `Result<T>`. In this case it is telling  the compiler that we are expecting an `[Drink]`. We can also easily add in other function that uses the same `enum` for `completion`.

```swift
func getCondiments(forDrink: Drink, completion: (Result<Condiment>) -> Void) {
    //some api call to get drinks from our server
}
```

### Adding a failure enum
In the project I worked on, I tend to add in a `failure` case into the `enum`.

```swift
enum Result<T> {
    case success(T)
    case failure(String)
    case error(Error)
}
```
Having just a `success` and `error` case can be insufficient to handle all condition of api call. Sometimes we have api call that did not error out but actually return a failure message due to user input. Such cases is not handle by `success(T)` and `error(Error)`. Therefore a additional `failure(String)` would help solve this issue.

```swift
func getCondiments(forDrink: Drink, completion: (Result<Condiment>) -> Void) {
    //some api call to get drinks from our server
    completion(.failure("Selected drink does not have any condiments"))
}
```

Using the example above, we can then show the message to the user like this:

```swift
func getCondiments(selectedDrink: Drink) {
    DrinkService().getCondiments(forDrink: selectedDrink) { result in
        switch result {
        case .success(let condiment):
            self.condiment = condiment
        case .failure(let message):
            showAlertView(message: message)
        case default
            showAlertView(message: error.localizedDescription)
        }
    }
}
```

This is just one of the many thing `enum` in `swift` empowered us to do. With the help of `generics` and `associated values`, we can design various `enum` to suit our need for different scenarios.
