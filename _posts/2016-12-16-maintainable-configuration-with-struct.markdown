---
layout: post
title:  "Maintainable configuration with struct!"
description: "Configuration pattern by using `struct`."
date:   2016-12-16 19:19:43
comments: true
keywords: "swift, struct, enum, ios, configs"
category: swift
tags:
- swift
- ios
---

Yes, you read it right. Although many times, we uses `enums` and `switch` but there is a much more maintainable and readable configuration pattern by using `struct`. I first read it here on [Jesse Squires blog](http://www.jessesquires.com/enums-as-configs/).

Very often, we use `enums` to filter different configuration in our code. I often use it in a custom `UITableViewCell` class.

Example, in a mobile commerce app, we have a `LineItemTableViewCell` that is displaying the status of the line item.

```swift
struct LineItem {
    let productName: String
    let price: Double
    let status: String
}
```

```swift
enum LineItemTableViewCellStyle {
    case cart
    case deliveryStatus
    case orderHistory
}
```

```swift
class LineItemTableViewCell: UITableViewCell {
    @IBOutlet weak var productNameLabel: UILabel!
    @IBOutlet weak var priceLabel: UILabel!
    @IBOutlet weak var orderStatusLabel: UILabel!

    func configure(lineItem: LineItem) {
        productNameLabel.text = lineItem.productName
        priceLabel.text = "\(lineItem.price)"
        orderStatusLabel.text = lineItem.status
    }

    func applyStyle(_ style: LineItemTableViewCellStyle) {
        switch style {
        case .cart:
            orderStatusLabel.textColor = UIColor.black
            orderStatusLabel.isHidden = true
            isUserInteractionEnabled = true
            accessoryType = .none
        case .deliveryStatus:
            orderStatusLabel.textColor = UIColor.red
            orderStatusLabel.isHidden = false
            isUserInteractionEnabled = true
            accessoryType = .disclosureIndicator
        case .orderHistory:
            orderStatusLabel.textColor = UIColor.blue
            orderStatusLabel.isHidden = false
            isUserInteractionEnabled = true
            accessoryType = .disclosureIndicator
        }
    }
}
```

The `LineItemTableViewCell` is being use by different `tableView`. Like `CartViewController`, `DeliveryStatusViewController` and `OrderHistoryViewController`. Each will call the `cell.applyStyle(style:)` for different style.

Let say we decide to allow user to review their order. We will add a  new `ReviewViewController` using `LineItemTableViewCell` as well. But we also have to add a new `.reviewOrder` into our `enum` and `switch`.

```swift
func applyStyle(_ style: LineItemTableViewCellStyle) {
    switch style {
    case .cart:
        orderStatusLabel.textColor = UIColor.black
        orderStatusLabel.isHidden = true
        isUserInteractionEnabled = true
        accessoryType = .none
    case .deliveryStatus:
        orderStatusLabel.textColor = UIColor.red
        orderStatusLabel.isHidden = false
        isUserInteractionEnabled = true
        accessoryType = .checkmark
    case .orderHistory:
        orderStatusLabel.textColor = UIColor.blue
        orderStatusLabel.isHidden = false
        isUserInteractionEnabled = true
        accessoryType = .disclosureIndicator
    case .reviewOrder:
        orderStatusLabel.textColor = UIColor.blue
        orderStatusLabel.isHidden = false
        isUserInteractionEnabled = true
        accessoryType = .disclosureIndicator
    }
}
```

This add more lines to the `applyStyle()` and become more difficult to read. We are just toggling the properties of the `UILabel` and cell. Sometimes I wonder if I could have a default style and just change those that need to be change. Thanks to Jesse, I learn a new and better way of doing it.

So instead of using `enum` and `switch` we can actually use `struct` to do it. The new `applyStyle()` will look like this:

```swift
func applyStyle(_ style: LineItemCellStyle) {
    orderStatusLabel.textColor = style.labelColor
    orderStatusLabel.isHidden = style.hideLabel
    isUserInteractionEnabled = style.allowInteraction
    accessoryType = style.accessoryType
}
```

The `LineItemCellStyle` is a `struct` that preset all the common pattern.

```swift
struct LineItemCellStyle {
    let labelColor: UIColor
    let hideLabel: Bool
    let allowInteraction: Bool
    let accessoryType: UITableViewCellAccessoryType

    init(labelColor: UIColor = UIColor.black, hideLabel: Bool = false, allowInteraction: Bool = true,
      accessoryType: UITableViewCellAccessoryType = .none) {
        self.labelColor = labelColor
        self.hideLabel = hideLabel
        self.allowInteraction = allowInteraction
        self.accessoryType = accessoryType
    }

    static var cartStyle: LineItemCellStyle {
        return LineItemCellStyle(hideLabel: true)
    }

    static var deliveryStyle: LineItemCellStyle {
        return LineItemCellStyle(labelColor: UIColor.red, allowInteraction: false, accessoryType: .checkmark)
    }

    static var historyStyle: LineItemCellStyle {
        return LineItemCellStyle(labelColor: UIColor.blue, accessoryType: .disclosureIndicator)
    }

    static var reviewStyle: LineItemCellStyle {
        return LineItemCellStyle(accessoryType: .disclosureIndicator)
    }
}
```

The advantage of this approach is a cleaner `LineItemTableViewCell`. We no longer have a massive `switch` statement but instead a custom `struct` to manage the different cell `style`. Default properties can also be preset in the `init` instead of settings it again and again.

If you notice that your custom `UITableViewCell` or any other `switch` and `enum` configuration is getting massively huge, consider this approach. I really like this and already refactor most of my project with this. It is a lot easier to write test for it as well!
