# RxSwift

Since we still mostly use Rx on our projects it's great to have the coding style as constistant as possible.

## Observables, drivers and the rest of the observable types

In most instances we want to ditch `observable` from our object names and treat these in the following way:

* Input that we get from the VC - `events`
* Output that we send into the VC from the presenter - `actions`

Not Preferred:

```swift
let buttonTapObservable: Observable<Void>
let itemsObservable: Observable<Void>
```

Preferred:

```swift
let buttonTap: Observable<Void>
let items: Observable<Void>
```

## Relays and subjects

An exception to this are `Relays` and `Subjects`. Since they aren't only meant to be read and can accept values, keeping `Relay` and `Subject` in the name helps to immediatelly recognize their context.

```swift
let modelUpdateSubject = BehaviorSubject<Model?>(value: nil)
let formEditsRelay = BehaviorRelay<Model>(value: model)
```

## Presenters' bind function

We should strive towards the exact same name for the bind function, along with its parameters. Please use the following in your projects:

```swift
presenter.configure(with output: Module.ViewOutput) -> Module.ViewInput
```

## Handle functions

Considering that all of the actions we need to handle will end up in the given `configure` function, we'll need to format the code appropriately so that we don't end up with a _"Massive View Controller"_ configure function.
The general template we follow is to use `handle` functions, which will transform our data and then either _subscribe_ or _return the value_. Example:

```swift
func configure(with output: Module.ViewOutput) -> Module.ViewInput {

    handle(action: output.action)

    let result = handle(action: output.action)

    return Module.ViewInput(
        result: result
    )
}
```

Depending on the action name and whether it conflicts with an already given name, for instance `handle(viewDidLoad: output.viewDidLoad)`, we can make use of labels to further identify what are we going to do in the `handle` function. In those instances, we would use the _viewActionWith_ label.

## Static transformation functions

Keep the blocks short and sweet. Ideally, a block should contain one or two lines of code that are descriptive about the task the block should end up doing.
In cases where more complex logic is required, it should be moved into dedicated functions.

Try to keep functions `pure` if possible.
A pure function is a function which preforms a transformation on the given input and produces an output.
It is expected that the involved function doesn't capture nor modify any state, only the given input.

That's why we're calling it `pure` — it only modifies what it gets without involving any usage of `self`.

By definition, such functions can and should be static. That helps the readers since they can be certain that there will be no side-effects, and that only the given input will be modified and returned. Making them static also helps us see potential errors through the compiler. _Code that tries to modify state in a static function will not build._

Example:

```swift
private var count: Int = 0
func increaseCount() -> Int
 ```

Not preferred:

 ```swift
func increaseCount() -> Int {
    return count += 1
}
let currentCount = increaseCount()
```

Preferred:

```swift
static func increaseCount(_ count: Int) -> Int {
    return count + 1
}
let currentCount = increaseCount(0)
```

To put this into perspective, when using the said example with Rx, we would end up doing something like this:

```swift
/*
Static functions that will drive and transform our data, without side-effects since `static`
will prevent that.
*/
static func createSession(withToken token: String) -> Session
static func storeUser(_ user: User)

/*
Closures are also great like the mentioned static functions above, with the only difference being
that they allow us to escape any parameters if we needed to do so.
*/
let createUser = { (username: String, password: String) -> Observable<User>
    return Observable.just(User(username: username, password: password))
}

createUser("Test", "infinum1")
    .flatMap(storeUser)
    .getToken()
    .flatMap(createSession(withToken:))
```

The goal here is to have the _call site_ of all the mutual functions down the line as clean as possible. That way, reading the entire Rx sequence is much simpler and it's overall easier to understand.

In contrast, doing it the _wrong_ way, we could end up with something like this:

```swift
createUser("Test", "infinum1")
    .flatMap { username, password in
        // A bunch
        // of
        // logic
        // to
        // create
        // a user
    }
    .getToken()
    .flatMap { token in
        // A bunch
        // of
        // logic
        // to
        // establish
        // a session
    }
```

By taking a look at the example above, you might see that not only does it end up being more difficult to read as the chain of operators expands, but it can also end up being tougher to understand since a lot of code can be chained through several blocks. This also isn't the _idiomatic way_ to write Rx and that's something we want to avoid as much as we can.

Furthermore, by taking a glance at the code, you can see what's happening in the block, but there's no context about the entirety of your logic—you need to interpret it yourself. That's just another reason why we need to strive towards named functions that immediately give details about their implementation.

## Capture lists

When writing Rx, you'll have to deal with and think about retain cycles. Capturing `self` in a block will usually result in a retain cycle if you aren't careful.
Generally, if you aren't sure whether your object will outlive the async operation, use `weak`.
If you're certain that the captured object will live until the operation is completed, use `unowned`.

A good rule of thumb is that, if you're using `disposeBag`, you're free to use `unowned` whenever you hook an Rx sequence to it. Why is that possible? Well, `diposeBag` usually lives in the object that initiates the sequence (in our case, it's mostly the _presenter_). When presenters ends up being disposed, the `disposeBag` gets disposed too, meaning that _they share their lifespan_.

In such cases, as you might've noticed already, it's fine to use `unowned` since we're certain that the `disposeBag` won't outlive the presenter—unless another object retained the `disposeBag`. Generally, you shouldn't be sending dispose bags between different objects unless you want to make your life harder. By all means, if it's needed, you're free to do it, but be mindful about it and leave a mark describing why it was done.

On the other hand, you might have a long-living operation that might continue even after the module closes. In that instance, it wouldn't be feasible to use `unowned`, as you'd only face a crash. Although such cases aren't really that common, they can happen.
Let's say that you initiate an upload on a certain screen and navigate out of it. In that case, you might want that signal to finish what it was doing and give feedback after it's done, regardless of whether it succeded or failed.

When dealing with `[weak self]` in blocks, you'll need to check whether `self` exists. Generally, the agreement of doing this is formatted as following:

```swift
apiCall.subscribe(onNext: { [weak self] model in
    guard let self = self else { return }
    // We're safe to use self here
})
```

For older versions of Swift, you'll need to escape self with \``backticks`\`. Other than that, the general usage is exactly the same:

```swift
apiCall.subscribe(onNext: { [weak self] model in
    guard let `self` = self else { return }
    // We're safe to use self here
})
```

In cases where you only ever need to escape a single property, you don't need to escape the _entire class_!  
For instance, you want to chain API calls. Further down the sequence you'll need to reference only the `interactor` to get the other call done. Another case would be that, after a certain action, you simply need to navigate using `wireframe`. These cases do not need entire `self` added to the capture list. Instead, you can do it like this:

```swift
interactor
    .initialApiCall()
    .flatMap { [weak interactor] data in
        return interactor?.chainedApiCall(data)
    }

input.openDetailsAction.subscribe(onNext: { [unowned wireframe, weak view] in
    view?.hideLoading()
    wireframe.navigate(to: .details)
})
```

>Note: If you have multiple objects in a capture list, take care that you always add `weak` or `unowned`. Take special care of that, since if you omit these, the object will end up being capture strongly!

Of course, if you need to capture several properties in that one particular block, it might be wiser to use `[weak/unowned self]` instead of making a long capture list with a dozen of properties.

---

Another important thing to mention about retaining `self` is that you're also able to do that _implicitly_. It's not obvious, it's easy to do, and it can be a complete hell to debug if
you don't really know what you're looking for. What does it mean to _capture self implicitly_? Well, let's look at the opposite first. For instance, a said block captures self _explicitly_:

```swift
interactor.apiCall()
    .flatMap {
        return self.interactor.anotherApiCall()
    }
```

We can easily fix this by making use of capture lists, by either using `weak` or `unowned`, depending on the case:

```swift
interactor.apiCall()
    .flatMap { [unowned self] in
        return self.interactor.anotherApiCall()
    }
```

Now - on the other hand, when talking about retaining `self` _implicitly_, we might write something like this:

```swift
class Presenter {

    // A function that takes an API model and returns some output afterwards
    func doWork(model: Model) -> Observable<Workload> { ... }

    func configure() {
        interactor.apiCall()
            .flatMap(doWork(model:))
            .subscribe()
            .disposed(by: disposeBag)
    }
}
```

At first or even at a second glance, this might look perfectly fine. However, we want to concentrate on this line `flatMap(doWork(model:))`. If you take a closer look at it, you might see that here we're calling a function that has an _inferred `self`_, that is, it belongs to the presenter. Even now, it might not look bad, since we're only passing a function, right?

But this actually causes a retain cycle.

If we were to write it without passing in the function:

```swift
apiCall().flatMap { model in
    return self.doWork(model: model)
}
```

As you can see, we're actually capturing `self` here and we're not adding it into a capture list, which in the end can and mostly will create a retain cycle.
Do note that this is actually the preferred way of making function calls, passing in functions to `maps` and `flatmaps` — however, not while they belong to an object, unless you're completely aware of what you're doing.

An easy fix and the proper way to do this is to make the actual transformation on the data without including `self`. Does this sound familiar? Well, it's good that it does because it should! We can simply make `func doWork(model: Model)` static and solve all of our problems.

In that case, we're not working with an object anymore, so we don't have to worry about causing any retain cycles. That transformation is also pure, which has already been highlighted in the previous part of the handbook.

```swift
func configure() {
    interactor.apiCall()
        .flatMap(Presenter.doWork(model:))
        .subscribe()
        .disposed(by: disposeBag)
}
```

> Note: Structs do not have to be escaped in the same manner as classes do because they're value types. In each case, the struct will end up being copied - which, depending on the case, might be a good or a bad thing.  
However, in cases where you end up using structs and not escaping self, a simple comment about it would be perfect so that people don't end up being confused over it.

## Dispose bag

In most instances you'll want to have a single dispose bag in the object that's setting up your Rx sequences. These can be either your Controllers, Presenters, Interactors, Cells - or any other class where you need to take care of resource disposal. Dispose bag is nice enough to take care of that for you.

To recap, the dispose bag will store all of the resources that you choose to dispose by calling `disposed(by: disposeBag)` on a `Disposable` object.

Disposable object will always be returned once you _subscribe_ to a particular Rx sequence.  
Dispose bag will store all the resources and, once it itself is about to be disposed, _it will clean up all of your resources beforehand_. In our case, when, for instance, your `Presenter` gets deallocated, it will end up pulling the `DisposeBag` with it (since it should be a property of the class) and cleaning everything up for you. Neat, right?

Note that we can also dispose the resources on our own by recreating the dispose bag object. When we assign a _new object_ to an _existing `DisposeBag`_ reference, it cleans up its resources and assigns the new object as the new `DisposeBag`. It's really that simple:

```swift
disposeBag = DisposeBag()
```

We can also do that by manually calling dispose on a _Disposable_ object, providing that we want it disposed right away:

```swift
apiCall()
    .flatMap { $0 }
    .subscribe()
    .dispose()
```

Both of these might end up being useful in cases where you absolutely want to free resources because of either intensive operations or something similar, but it isn't something that you'll do that often. However...

It gets a bit different when used with _cells_. Since cells are reused, resources need to be disposed a lot more frequently. If a cell gets reused and resources are still kept in either the cell or its item, it might (and most likely will) end up showing the wrong data.  
In order to prevent that, _we want to dispose of the resources related to the cell and recreate the dispose bag when the cell will end up being reused_.

Thankfully, we have a lifecycle function just for that:

```swift
override func prepareForReuse() {
    super.prepareForReuse()
    disposeBag = DisposeBag()
}
```

Items shouldn't really be storing and keeping a `DisposeBag` unless it's absolutely necessary. Rather, they should only transform their actions and create presentable data that needs to be sent to the cell, subscribed onto and disposed by the cells' `DisposeBag`.

And last but not least - it's preferrable that each of your individual objects has its appropriate `DisposeBag`. If for some reason that isn't possible and you want to send `DisposeBag` as a dependency, think about it once again. If it's possible to avoid, it would definitely be a wiser design choice, _since that might end up with `DisposeBag` being **captured**_. That means that none of your resources will end up being disposed.

If you do opt into sending the `DisposeBag`, be very careful of its usage and managing capture lists in all instances where the dispose bag is being used.

In this particular case, also be mindful of the implicit retain cycles which we've covered in an earlier point in the handbook.

## Side-effects

Now, we've mentioned that handling side-effects should be kept out of the transformation functions in order to keep them pure. That also applies to any blocks that you might open in a Rx sequence altogether.

However, it's completely understandable that you cannot simply ignore side-effects altogether and work around them. They should generally never be used inside the regular operators that we use to transform values, since it makes the logic more _ambiguous_ and harder to track down bugs.

Rx has a solution to that and you should be using it whenever you need to create a side-effect, at any part of the sequence. So, if we focus on rights and wrongs:

Not preferred:

```swift
apiCall().map { [unowned self] model in
    let items = makeItems(from: model)
    self.items = items
    return items
}
```

Preferred:

```swift
apiCall
    .map(makeItems(from:))
    .do(onNext: { [unowned self] items in
        self.items = items
    })
```

The `do` operator is precisely what you want. It's also great in the sense that you can hook it up after any part of the sequence.
We could've also added it before the map to store the `model` that we've retrieved from the API, for instance.

If you can, though, try to limit yourself in using side-effects to only those instances when they're really needed. It makes the code easier to understand if you're reading it from top to bottom, without having to backtrack to see what happens with variables that have been changed when you're already half through your sequence. Well, that and the fact that one of the main benefits of Rx is to avoid side-effects.

## Subjects and Relays

Subjects are great and we'll often need to write something into a sequence rather than only read from them, but there are a few things to keep in mind here. Most notably, not overusing subjects or relays in places where you can get away without them.
The place where you'll usually want a `Subject` or a `Relay`:

* When you want to bridge from a non-reactive code into reactive, e.g., forwarding a delegate callback into a sequence of observables
* When you need both reading and writing, e.g., an object sent from a different module through which you want to signal a change in the owner module
* To represent sort of _state_ for UI components or similar

Do note that if you can get away with a `Driver` or an `Observable`, which means that you only ever want to read data, this is the way to go. _Do not introduce `Relays` or `Subjects` in places where they aren't really needed—that only leaves a gateway for potential bugs._

Unlike other observable types, we've mentioned that we'll actually keep _-relay_ or _-subject_ in a variable name, so keep that in mind when declaring them.

```swift
func configure(with viewOutput: View.Output) -> View.Input {
    let modelRelay: BehaviorRelay<Model> = BehaviorRelay(value: Model.default)
    handle(action, with: modelRelay)
}
```

Use subjects or relays within the scope of either the `configure` or `handle` functions! That way, you're limiting their scope to that particular function and also limiting anyone from altering their state throughout the entire class. If a wrong mutation occurs, you can just glance over the function see where it happened in a much easier fashion.

Last, but not least, if you need to pass subjects or relays as mentioned, _do not store them as properties in child modules_ because that can result with memory leaks. It isn't a big leak and is generally related to how RxSwift manages its resources, but depending on what you're passing through the subject or a relay, it might end up being nasty. This will be covered in more detail in the _Common issues_ section.

## Action Relay

Have you ever worked with a `Switch` control that needs to trigger an API call? Did you need to maintain the state of the switch depending on the result of that call?

If you did, you'll most certainly know that it can get annoying easily and quickly. The general use case follows:

* Interact with a switch
* Switch triggers an API call that returns a result
* Result interacts with the switch again
* Switch triggers an API call that returns a result
* ...

Essentially, we've just created a fantastic infinite loop of API call triggering switch that'll ruin our life. This is where the `ActionRelay` or `Bidirectional{Publish,Behavior}Relay` comes in place.

As the name suggests, the relay in use here actually has a _two-way-binding_ instead of the traditional relay behavior. Now, at first, you might be thinking that this will do the exact same thing as the scenario that has initially been described, which is definitely a valid concern. However...

In this instance, you need to notice that in both cases we're using `ControlProperty`. If you dig deeper into its behavior you might notice that `ControlProperty` won't actually emit an event _**unless**_ it receives an actual _`UIControlEvent`_, which in this case it won't. We're assigning value directly to the property itself, which keeps us safe from an infinite loop!

Now, we can simply bind both our _Switch_ and the _Relay_ together and have them cooperate depending on the changes between the two. A signal will adjust the value of the switch control whilst the switch control will then again only fire if it receives an actual change.

>Note: By calling the `setOn(_: Bool, animated: Bool)` function programatically, switch will once again call the _valueChanged_ event causing the issue.

```swift
class BidirectionalPublishRelay<T> {

    private let workInput = PublishRelay<T>()
    private let viewInput = PublishRelay<T>()

    var workProperty: ControlProperty<T> {
        return ControlProperty<T>(
            values: viewInput.asObservable(),
            valueSink: Binder(self) { (_, input) in
                // Need to retain self here
                self.workInput.accept(input)
            }
        )
    }

    var viewProperty: ControlProperty<T> {
        return ControlProperty<T>(
            values: workInput.asObservable(),
            valueSink: Binder(self) { (_, input) in
                // Need to retain self here
                self.viewInput.accept(input)
            }
        )
    }
}

class BidirectionalBehaviorRelay<T> {

    private let workInput: BehaviorRelay<T>
    private let viewInput = PublishRelay<T>()

    init(value: T) {
        workInput = BehaviorRelay(value: value)
    }

    var workProperty: ControlProperty<T> {
        return ControlProperty<T>(
            values: viewInput.asObservable(),
            valueSink: Binder(self) { (_, input) in
                // Need to retain self here
                self.workInput.accept(input)
            }
        )
    }

    var viewProperty: ControlProperty<T> {
        return ControlProperty<T>(
            values: workInput.asObservable(),
            valueSink: Binder(self) { (_, input) in
                // Need to retain self here
                self.viewInput.accept(input)
            }
        )
    }
}
```
