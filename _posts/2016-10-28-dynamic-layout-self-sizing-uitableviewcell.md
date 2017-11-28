---
layout: post
title: "Dynamic layout - Self Sizing UITableViewCell"
date: 2016-10-28 21:17:31
comments: true
description: "How to easily create a dynamic UITableViewCell using constraint set in Storyboard that have a UIImageView and UILabel. These details could possibly be updated from an api call like most app out there."
keywords: "swift, uitableview, uitableviewcell, ios"
category: swift
tags:
- swift
- ios
---

How to easily create a dynamic `UITableViewCell` using `constraint` set in `Storyboard` that have a `UIImageView` and `UILabel`. These details could possibly be updated from an api call like most app out there.

## Overview
We will be creating a simple coffee apps that explains the different kind of coffees. The app will have a coffee image and its descriptions in each `UITableViewCell`. The images will be in different sizes and the number of characters in descriptions will be different as well.

A few screenshots showing the coffees app:

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/1/ss1.png" width="220">
<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/1/ss2.png" width="220">
<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/1/ss3.png" width="220">

### Setup Constraint
Create a custom `UITableViewCell` with a `UIImageView` and `UILabel` in it.

Set constraints for `UIImageView` base on the screenshot below.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/1/imageView.png" width="800">

And follow the following screenshot for `UILabel`.

<img src="https://project-zeta.s3-ap-southeast-1.amazonaws.com/post/1/label.png" width="800">

### Configure `UITableView`
Next, we will need to let `UITableView` know that we want the `rowHeight` to be automatic determined.

```swift
override func viewDidLoad() {
    super.viewDidLoad()
    tableView.rowHeight = UITableViewAutomaticDimension
    tableView.estimatedRowHeight = 140
}
```
 `estimatedRowHeight` not only help with the improve of performance when loading `tableView` but it also need to be set to any value bigger than `0`. By default the value is `0` which means there is no estimate.

 A more details description from [Apple](https://developer.apple.com/reference/uikit/uitableview/1614925-estimatedrowheight):

 >Providing a nonnegative estimate of the height of rows can improve the performance of loading the table view. If the table contains variable height rows, it might be expensive to calculate all their heights when the table loads. Using estimation allows you to defer some of the cost of geometry calculation from load time to scrolling time.

>When you create a self-sizing table view cell, you need to set this property and use constraints to define the cell’s size.

>The default value is 0, which means there is no estimate.

### Custom class: `Coffee` and `CoffeeTableViewCell`
Next we will create a custom `CoffeeTableViewCell` class and `Coffee` struct. Our `Coffee` struct will be holding a `imageName` and `description` of the coffee.

```swift
struct Coffee {

    let imageName: String
    let description: String

    init(imageName: String, description: String) {
        self.imageName = imageName
        self.description = description
    }
}
```

In `storyboard`, update the cell to our custom class `CoffeeTableViewCell`. Then create an `@IBOulet` for the `UIImageView ` and `UILabel `.

In the `CoffeeTableViewCell` we will create a `configure` function to update its `IBOutlet` with a `Coffee` model.

```swift
class CoffeeTableViewCell: UITableViewCell {

    @IBOutlet weak var coffeeImageView: UIImageView!
    @IBOutlet weak var coffeeDescLabel: UILabel!

    func configure(coffee: Coffee) {
        coffeeImageView.image = UIImage(named: coffee.imageName)
        coffeeDescLabel.text = coffee.description
    }
}
```

### In our controller `CoffeesViewController`
We will have an `@IBOulet tableView` hookup from `storyboard`. We will also add `getCoffeeData()` at the end of `viewDidLoad()`

```swift
class CoffeesViewController: UIViewController {
    @IBOutlet weak var tableView: UITableView!
    let coffeeService = CoffeeService()
    var coffees = [Coffee]()

    override func viewDidLoad() {
        super.viewDidLoad()
        tableView.rowHeight = UITableViewAutomaticDimension
        tableView.estimatedRowHeight = 140

        getCoffeeData()
    }

    func getCoffeeData() {
        coffeeService.getCoffeeData() { coffees in
            self.coffees = coffees
            self.tableView.reloadData()
        }
    }
}

```
`CoffeeService` is just a fake service class with a completion `func` that return `[Coffee]` like it is being fetched from a api call.

Lastly, setup the `datasource` with the required `func`. Remember to hook up the `delegate` and `datasource` from `tableView` to `CoffeesViewController` in the `storyboard`.

```swift
extension CoffeesViewController: UITableViewDataSource {

    func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return coffees.count
    }

    func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "CoffeeTableViewCell") as! CoffeeTableViewCell
        let coffee = coffees[indexPath.row]
        cell.configure(coffee: coffee)

        return cell
    }
}
```

## Sample Project
A sample project could be download [here](https://github.com/Zaccc123/coffees).

Thats all you need to setup a self sizing `UITableViewCell`. Ensure that that the correct `constraint` is set in `storyboard`, `rowHeight` and `estimatedRowHeight` is also updated for `tableView`.
