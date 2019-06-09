---
layout: post
title:  "a dead simple solution to lift up errors in ios"
categories: 
---

In this post, I want to show you how you can utilize [the responder chain](https://developer.apple.com/documentation/uikit/touches_presses_and_gestures/using_responders_and_the_responder_chain_to_handle_events?language=objc) to bubble up an error message through your app UI structure till it gets to the node which can handle it. It is especially useful if you are a fan of [Container View Controller](https://developer.apple.com/library/archive/featuredarticles/ViewControllerPGforiPhoneOS/ImplementingaContainerViewController.html).
Responder Chain is a linked list built and maintained by iOS that represents the prioritized list of object that potentially can respond to events that occur in UI. If you like to learn more about Responder Chain pattern in general [here](https://refactoring.guru/design-patterns/chain-of-responsibility) or more specific in iOS [here](https://useyourloaf.com/blog/using-the-responder-chain/) and [here](https://swiftrocks.com/understanding-the-ios-responder-chain.html).

Suppose that you have a nested structure of views and view controllers that presents the UI of your app. If something goes wrong in any of this view controllers or views and you don't want to handle the error there and you prefer you pass it up to its ancestors, you might use delegate pattern or closure or other solution. But you usually end up adding some kind of one-direction wire between views and view controllers which have parent-child relation. Fortunately, we already have this one-direction wire between parent and child for free and we just need to use it.

Here is what you can do:
``` swift
extension UIResponder {
    @objc
    func showError(message: String) {
        self.next?.showError(message: message)
    }
}
```

Here we add a method to `UIResponder` that consequently, this method will be available on any UIView, UIViewController, UIWindow or UIApplication. What it does is really simple, it passes the error to the ancestor of the current UIResponder object. If we override this method in a subclass of UIResponder, we will be able to catch the error and handle it however we like.

You might use a more sophisticated parameter for `showError`. For example, you can pass an [Error](https://developer.apple.com/documentation/swift/error) instance, so that an UIResponder can handle just some type of error and passes those that don't handle to its ancestor.

Here is an example:

``` swift
class WhiteViewController: UIViewController {
    var errorLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .white
        
        // Add a child view controller
        let childVC = RedViewController()
        childVC.view.frame = vc.view.bounds.insetBy(dx: 50, dy: 50)
        addChild(childVC)
        view.addSubview(childVC.view)
        childVC.didMove(toParent: self)
        
        // Add a label for showing error messages
        errorLabel = UILabel(frame: .init(x: 0, y: 0, width: view.frame.width, height: 40))
        view.addSubview(errorLabel)
    }
    
    override func showError(message: String, fatal: Bool) {
        // Show the error message and clear it after 1sec
        errorLabel.text = message
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
            self?.errorLabel.text = ""
        }
    }
}

class RedViewController: UIViewController {
    var errorLabel: UILabel!
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .red
        
        // Add a child view controller
        let childVC = GreenTableViewController()
        childVC.view.frame = view.bounds.insetBy(dx: 50, dy: 50)
        addChild(childVC)
        view.addSubview(childVC.view)
        childVC.didMove(toParent: self)
        
        // Add a label for showing error messages
        errorLabel = UILabel(frame: .init(x: 0, y: 0, width: view.frame.width, height: 40))
        view.addSubview(errorLabel)
    }
    
    override func showError(message: String, fatal: Bool) {
        if fatal {
            // Show the error message and clear it after 1Sec
            errorLabel.text = message
            DispatchQueue.main.asyncAfter(deadline: .now() + 1) {  [weak self] in
                self?.errorLabel.text = ""
            }
        } else {
            // Pass the error to the ancestor
            next?.showError(message: message, fatal: fatal)
        }
    }
}

class SimpleCellView: UITableViewCell {
    var fatal: Bool!
    
    override func setSelected(_ selected: Bool, animated: Bool) {
        super.setSelected(selected, animated: animated)
        if selected {
            showError(message: "Something went wrong!", fatal: fatal)
        }
    }
}

class GreenTableViewController: UITableViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        view.backgroundColor = .green
        
        self.tableView.register(SimpleCellView.self, forCellReuseIdentifier: "SimpleCellView")
    }
    
    override func numberOfSections(in tableView: UITableView) -> Int {
        return 1
    }
    
    override func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
        return 10
    }
    
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "SimpleCellView") as? SimpleCellView
        cell?.textLabel?.text = "Cell #\(indexPath.row)"
        cell?.fatal = indexPath.row % 2 == 0
        return cell!
    }
}

extension UIResponder {
    @objc
    func showError(message: String, fatal: Bool) {
        self.next?.showError(message: message, fatal: fatal)
    }
}
```

This is the responder chain of this example:
```
Responder Chain:
 
UIApplication (next: nil)
 |
 |__ UIWindow (next: UIApplication)
      |
      |__ WhiteViewController (next: UIWindow)
            |
            |__ RedViewController (next: WhiteViewController)
                 |
                 |__ GreenTableViewController (next: RedViewController)
                      |
                      |__ SimpleCellView #1  (next: GreenTableViewController)
                          SimpleCellView #2  (next: GreenTableViewController)
                          SimpleCellView #3  (next: GreenTableViewController)
                          SimpleCellView #4  (next: GreenTableViewController)
                          SimpleCellView #5  (next: GreenTableViewController)
                          SimpleCellView #6  (next: GreenTableViewController)
                          SimpleCellView #7  (next: GreenTableViewController)
                          SimpleCellView #8  (next: GreenTableViewController)
                          SimpleCellView #9  (next: GreenTableViewController)
                          SimpleCellView #10 (next: GreenTableViewController)
```

![](https://github.com/coybit/coybit.github.io/raw/master/assets/responder-chain-ui.gif)

In `SimpleCellView` when user selects a cell, `showError` is called:
``` swift
override func setSelected(_ selected: Bool, animated: Bool) {
    super.setSelected(selected, animated: animated)
    if selected {
        showError(message: "Something went wrong!", fatal: fatal)
    }
}
```

Because we haven't overriden `showError` in `SimpleCellView`, the default implementation is called:
``` swift
func showError(message: String, fatal: Bool) {
    self.next?.showError(message: message, fatal: fatal)
}
```

The next item in the chain is `GreenTableViewController` which doesn't have an overridden version of `showError`, so the default implementation is called again and the next item in the responder chain is `RedViewController`. It has an implementation of `showError` but catches and handles the error which is fatal, otherwise passes it to its ancestor which in this case is `WhiteViewController`:

``` swift
override func showError(message: String, fatal: Bool) {
    if fatal {
        // Show the error message and clear it after 1Sec
        errorLabel.text = message
        DispatchQueue.main.asyncAfter(deadline: .now() + 1) {  [weak self] in
            self?.errorLabel.text = ""
        }
    } else {
        // Pass the error to the ancestor
        next?.showError(message: message, fatal: fatal)
    }
}
```

Those errors which are not fatal get to `WhiteViewController` and it handles them all equally:
``` swift
 override func showError(message: String, fatal: Bool) {
    // Show the error message and clear it after 1sec
    errorLabel.text = message
    DispatchQueue.main.asyncAfter(deadline: .now() + 1) { [weak self] in
        self?.errorLabel.text = ""
    }
}
```
