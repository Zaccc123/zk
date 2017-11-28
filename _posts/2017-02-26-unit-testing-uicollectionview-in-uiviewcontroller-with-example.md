---
layout: post
title: "Unit Test - UICollectionView in UIViewController with example"
description: "Testing is an important aspect of development. No matters if you belong to the Test Driven Development(TDD) group or not, one should never ignore tests."
date: 2017-02-26 16:39:18
comments: true
keywords: "swift, testing uitest, view, controller, ios, tdd"
category: swift
tags:
- swift
- ios
---

Testing is an important aspect of development. No matters if you belong to the Test Driven Development(TDD) group or not, one should never ignore tests.

Beginning from Xcode 5, Apple introduce the XCTest Framework. Performance measurement was then introduce on Xcode 6 and UI Testing on Xcode 7. However, unit testing a `UIViewController` is still difficult. This is mainly due to the lifecycle and the way the app module is setup.

### Unit test or UITest `UIViewController`?
Unit test aim to test small portions of our code during development while UI Test tend to only be implemented at the end like a integration test. Also, unit test run a lot faster compare to UI test. I would say, you need both to have a more robust code base.

### But unit testing `UIViewController` is difficult
Yes, is difficult. We have lot of things to take care of before having a suitable test environment. However, difficult does not equate to undoable. We can still write tests or even do TDD for it. These would also help us to understand the lifecycle of `UIViewController` better.

## Usual Set up
This is what I have in most of my `UIViewController` tests.

```swift
import UIKit
import XCTest
@testable import MyPhotoProject

class PhotoViewControllerTests: XCTestCase {

    var viewController: PhotoViewController!

    override func setUp() {
        super.setUp()
        //1
        let storyboard = UIStoryboard(name: "Main", bundle: Bundle.main)
        let navigationController = storyboard.instantiateInitialViewController() as! UINavigationController
        viewController = navigationController.topViewController as! PhotoViewController

        //2
        UIApplication.shared.keyWindow!.rootViewController = viewController

        //3
        XCTAssertNotNil(navigationController.view)
        XCTAssertNotNil(viewController.view)
    }
}
```
**Number 1** is the setup to create the `ViewController` that we want to test. If on another app that are using `UITabBarController`, we can replace the `navigationController` with it. We can also use `instantiateViewController(withIdentifier:)` if needed. I usually use a framework called [R.swift](https://github.com/mac-cain13/R.swift). It very handy to access `viewController`, `image` or `identifier` that is already in the project.

**2** just like what our `AppDelegate` do, we set it to the `rootViewController` when the setup is run. This is really important if we are testing views related stuff like `UICollectionView`.

**3** help us to prepare the views with `IBOutlet` and `IBAction` connection that is setup via storyboard. I usually use `_ = controller.View` but have recently change to these after reading it on NatashaTheRobot blog [post](https://www.natashatherobot.com/ios-testing-view-controllers-swift/).

### Testing a UICollectionView
So with the above setup, a sample test of `UICollectionView` in a `ViewController` is as follows:

```swift
func testCollectionViewCellsIsDisplayedWithMatchingImage() {
    //1 Arrange
    let fakeImagesName = ["FakeA", "FakeB", "FakeC"]
    viewController.imagesName = fakeImagesName

    //2 Act
    viewController.collectionView.reloadData()
    RunLoop.main.run(until: Date(timeIntervalSinceNow: 0.5))

    //3 Assert
    let cells = viewController.collectionView.visibleCells as! [PhotoCollectionViewCell]
    XCTAssertEqual(cells.count, fakeImagesName.count, "Cells count should match array.count")
    for I in 0...cells.count - 1 {
        let cell = cells[I]
        XCTAssertEqual(cell.photoImageView.image, UIImage(named: fakeImagesName[I]), "Image should be matching")
    }}
```

At **1**, we **Arrange** the fake data required for the test.

**2**, we **Act** on it. We called the `reloadData()` to trigger the `collectionView` delegate. The `RunLoop.main.run(until: Date(timeIntervalSinceNow: 0.5))` is required as a "hacky" way of waiting for the data to be loaded.

We then **Assert** on **3**. These is where all the checking is done. Here, we ensure that the count on the `cells` is correct and each `cell.photoImageView.image` is equal to the `UIImage` that it should be displaying.

### Quick and Nimble
[Quick](https://github.com/Quick/Quick) are a testing framework framework for `Swift` and `Objective-C`. Using it together with [Nimble](https://github.com/Quick/Nimble), a matcher framework would help us to write better tests and have less boilerplate code.

With the above example, the test would look like this with `Quick` and `Nimble`.

```swift
class PhotoViewControllerSpec: QuickSpec {
    override func spec() {
        describe("A PhotoViewController") {
            var viewController: PhotoViewController!

            beforeEach {
                let storyboard = UIStoryboard(name: "Main", bundle: Bundle.main)
                let navigationController = storyboard.instantiateInitialViewController() as! UINavigationController
                viewController = navigationController.topViewController as! PhotoViewController

                let window = UIWindow(frame: UIScreen.main.bounds)
                window.makeKeyAndVisible()
                window.rootViewController = viewController

                _ = viewController.view
            }
            context("should be displaying photo on cells") {
                it("should match datasource count and display correct UIImage") {
                    let fakeImagesName = ["coffee1","coffee2","coffee3","coffee4","coffee5","coffee6"]
                    viewController.imagesName = fakeImagesName
                    viewController.collectionView.reloadData()

                    waitUntil { done in
                        if let cells = viewController.collectionView.visibleCells as? [PhotoCollectionViewCell] {
                            expect(cells).to(haveCount(fakeImagesName.count))
                            for i in 0...cells.count - 1 {
                                let cell = cells[i]
                                expect(cell.photoImageView.image).to(equal(UIImage(named: fakeImagesName[i])))
                            }
                            done()
                        }
                    }
                }
            }
        }
    }
}
```
Using `Quick` and `Nimble` helps us to reduce the **Arrange** code significantly as the `ViewController` get larger. It also help us to write better tests with `expect` instead of `XCTAssert`.

In the case above, instead of using `RunLoop`, we can use the `waitUntil` for the action to complete. We also replace all the `XCTAssert` with `expect`. Looking at the `expect(cells).to(haveCount(fakeImagesName.count))`, it read very much like a English sentence.

### Conclusion
Unit testing a `UIViewController` is certainly doable. As times goes by, Apple would surely improve the `XCTest Framework`. Covering our code with more tests cases would surely help make the app more robust.
