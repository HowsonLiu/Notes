# RxSwift 原理
从一段很简单的代码开始
```swift
// 1
let observable = Observable<Any>.create { observer -> Disposable in
    observer.onNext("hello")
    observer.onCompleted()
    return Disposables.create()
}
// 2
observable.subscribe(onNext: { text in
    print("recv \(text)")
}, onCompleted: {
    print("completed")
})
// 3
```
发出一个事件有很多种方法，我们选择最简单的`Observable.create`方法，它接受一个回调函数，用这个回调函数创建出一个序列  

我们需要思考以下问题  
- 序列的事件是什么时候发送的？(1 or 2 or 3)
- 观察者的代码是什么时候执行的？(1 or 2 or 3)

## Observable.create
```swift
extension ObservableType {
    public static func create(_ subscribe: @escaping (AnyObserver<Element>) -> Disposable) -> Observable<Element> {
        // 看这里↓
        return AnonymousObservable(subscribe)
    }
}
```
create的代码很简单，他把功能转到`AnonymousObservable`中实现了
```swift
final private class AnonymousObservable<Element>: Producer<Element> {
    typealias SubscribeHandler = (AnyObserver<Element>) -> Disposable

    let _subscribeHandler: SubscribeHandler     // 看这里

    init(_ subscribeHandler: @escaping SubscribeHandler) {
        self._subscribeHandler = subscribeHandler   // 看这里
    }

    override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
        let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
        let subscription = sink.run(self)
        return (sink: sink, subscription: subscription)
    }
}
```
`AnonymousObservable`存下了创建序列的回调，并在后续的`run`方法使用到了他

## Observable.subscribe
```swift
    public func subscribe(onNext: ((Element) -> Void)? = nil, onError: ((Swift.Error) -> Void)? = nil, onCompleted: (() -> Void)? = nil, onDisposed: (() -> Void)? = nil)
        -> Disposable {
            let disposable: Disposable
            
            if let disposed = onDisposed {
                disposable = Disposables.create(with: disposed)
            }
            else {
                disposable = Disposables.create()
            }
            
            #if DEBUG
                let synchronizationTracker = SynchronizationTracker()
            #endif
            
            let callStack = Hooks.recordCallStackOnError ? Hooks.customCaptureSubscriptionCallstack() : []
            // 看这里↓
            let observer = AnonymousObserver<Element> { event in
                
                #if DEBUG
                    synchronizationTracker.register(synchronizationErrorMessage: .default)
                    defer { synchronizationTracker.unregister() }
                #endif
                
                switch event {
                case .next(let value):
                    onNext?(value)
                case .error(let error):
                    if let onError = onError {
                        onError(error)
                    }
                    else {
                        Hooks.defaultErrorHandler(callStack, error)
                    }
                    disposable.dispose()
                case .completed:
                    onCompleted?()
                    disposable.dispose()
                }
            }
            return Disposables.create(
                self.asObservable().subscribe(observer),
                disposable
            )
    }
```
总的来说，`subscribe`函数创建了一个`AnonymousObserver`，在他的事件处理函数里面对四个回调进行分发处理。并且后续调用了`self.asObservable().subscribe(observer)`  
从这里能看得出来，示例代码都是方便我们使用的扩展，其本质是创建出`AnonymousObservable`与`AnonymousObserver`，并且使用`AnonymousObservable.subscribe(observer:)`使他们链接，所以我们后续关注这三个点就好了  

## AnonymousObserver
观察者，他的作用就是对不同的事件进行处理，可以简单理解成按钮的点击事件。  
因此他的实现也特别简单，我们从他的继承链出发，剖析他的作用：  
`AnonymousObserver` -> `ObserverBase` -> `Disposable` & `ObserverType`  

1. `Disposable` & `ObserverType`  
    ```swift
    public protocol Disposable {
        func dispose()      // 看这里
    }

    public protocol ObserverType {
        associatedtype Element

        @available(*, deprecated, renamed: "Element")
        typealias E = Element

        func on(_ event: Event<Element>)        // 看这里
    }
    ```
    `Disposable`就是RAII，用变量的生命周期绑定回调事件生效的生命周期；`ObserverType`定义了何为观察者，观察者必须实现其`on`函数，对不同的事件进行处理  
2. `ObserverBase`
    ```swift
    class ObserverBase<Element> : Disposable, ObserverType {
        private let _isStopped = AtomicInt(0)
        
        func on(_ event: Event<Element>) {      // 看这里
            switch event {
            case .next:
                if load(self._isStopped) == 0 {
                    self.onCore(event)
                }
            case .error, .completed:
                if fetchOr(self._isStopped, 1) == 0 {
                    self.onCore(event)
                }
            }
        }

        func onCore(_ event: Event<Element>) {
            rxAbstractMethod()
        }

        func dispose() {
            fetchOr(self._isStopped, 1)
        }
    }
    ```  
    `ObserverBase`简单实现了两个协议，并定义了一个原子变量，标识事件之后可否继续，对事件上报到`onCore`函数处理  
3. `AnonymousObserver`
    ```swift
    final class AnonymousObserver<Element>: ObserverBase<Element> {
    typealias EventHandler = (Event<Element>) -> Void
    
    private let _eventHandler : EventHandler        // 看这里
    
    init(_ eventHandler: @escaping EventHandler) {
    #if TRACE_RESOURCES
            _ = Resources.incrementTotal()
    #endif
            self._eventHandler = eventHandler
        }

        override func onCore(_ event: Event<Element>) {     // 看这里
            return self._eventHandler(event)
        }
        
    #if TRACE_RESOURCES
        deinit {
            _ = Resources.decrementTotal()
        }
    #endif
    }
    ```
    `AnonymousObserver`保存用户传进来的`eventHandler`，并于`onCore`函数交予其处理  

总结：`AnonymousObserver`保存用户的事件处理函数，并用`Dispose`管理生命周期  

## `AnonymousObservable`
序列，也称为被观察者，他的作用就是产生一系列的事件，并且可被观察者订阅  
他的继承链如下：  
`AnonymousObservable` -> `Producer` -> `Observable` -> `ObservableType`  
1. `ObservableType`
    ```swift
    public protocol ObservableType: ObservableConvertibleType {
        // 看这里
        func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element
    }

    extension ObservableType {
        public func asObservable() -> Observable<Element> {
            return Observable.create { o in
                return self.subscribe(o)
            }
        }
    }
    ```
    `ObservableType`定义了何谓序列：可以被订阅的称之为序列。因此需要自行实现`subscribe`函数
2. `Observable`
    ```swift
    public class Observable<Element> : ObservableType {
    init() {
    #if TRACE_RESOURCES
        _ = Resources.incrementTotal()
    #endif
        }
        
        public func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            rxAbstractMethod()
        }
        
        public func asObservable() -> Observable<Element> {
            return self
        }
        
        deinit {
    #if TRACE_RESOURCES
            _ = Resources.decrementTotal()
    #endif
        }
    }
    ```
    `Observable`只实现了`asObservable`，返回为self，其他的并没有处理
3. `Producer`
    ```swift
    class Producer<Element> : Observable<Element> {
        override init() {
            super.init()
        }

        override func subscribe<Observer: ObserverType>(_ observer: Observer) -> Disposable where Observer.Element == Element {
            if !CurrentThreadScheduler.isScheduleRequired {
                let disposer = SinkDisposer()
                let sinkAndSubscription = self.run(observer, cancel: disposer)  // 看这里
                disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

                return disposer
            }
            else {
                return CurrentThreadScheduler.instance.schedule(()) { _ in
                    let disposer = SinkDisposer()
                    let sinkAndSubscription = self.run(observer, cancel: disposer)  // 看这里
                    disposer.setSinkAndSubscription(sink: sinkAndSubscription.sink, subscription: sinkAndSubscription.subscription)

                    return disposer
                }
            }
        }

        func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
            rxAbstractMethod()
        }
    }
    ```
    `Producer`实现了`subscribe`函数，他在里面判断了一下是否需要切换线程，然后将`observer`作为参数，委托到`run`函数处理。同时他自己声明了`run`函数，但保留实现
4. `AnonymousObservable`  
    ```swift
    final private class AnonymousObservable<Element>: Producer<Element> {
        typealias SubscribeHandler = (AnyObserver<Element>) -> Disposable

        let _subscribeHandler: SubscribeHandler

        init(_ subscribeHandler: @escaping SubscribeHandler) {
            self._subscribeHandler = subscribeHandler
        }

        override func run<Observer: ObserverType>(_ observer: Observer, cancel: Cancelable) -> (sink: Disposable, subscription: Disposable) where Observer.Element == Element {
            let sink = AnonymousObservableSink(observer: observer, cancel: cancel)  // 看这里***
            let subscription = sink.run(self)   // 看这里***
            return (sink: sink, subscription: subscription)
        }
    }
    ```
    `AnonymousObservable`作为其中一种序列，他选择了使用回调的方式创建序列，因此他保存了用户传进来的`subscribeHandler`，同时他实现了`run`函数，在内部使用了`AnonymousObservableSink`对observer进行处理

总结：`AnonymousObservable`是一个序列，他内部拥有切换线程的逻辑，他实现了`subscribe`函数，不过具体实现交给了`AnonymousObservableSink.run`处理  

## `AnonymousObservableSink`
终于来到了`AnonymousObservableSink`，上面的类实际上都是回调的封装，还加上一点线程切换以及生命周期的管理，真正决定运行时机的`subscribe`原理，就隐藏在这个类中  
先看看他的继承链：  
`AnonymousObservableSink` -> `ObserverType` & `Sink`  
`Sink` -> `Disposable`  
先记住，下面我们直接看他们的源码
```swift
final private class AnonymousObservableSink<Observer: ObserverType>: Sink<Observer>, ObserverType {
    typealias Element = Observer.Element 
    typealias Parent = AnonymousObservable<Element>

    // state
    private let _isStopped = AtomicInt(0)

    #if DEBUG
        private let _synchronizationTracker = SynchronizationTracker()
    #endif

    override init(observer: Observer, cancel: Cancelable) {
        super.init(observer: observer, cancel: cancel)
    }

    func on(_ event: Event<Element>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        switch event {
        case .next:
            if load(self._isStopped) == 1 {
                return
            }
            self.forwardOn(event)
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.forwardOn(event)
                self.dispose()
            }
        }
    }

    func run(_ parent: Parent) -> Disposable {
        return parent._subscribeHandler(AnyObserver(self))
    }
}
```
```swift
class Sink<Observer: ObserverType> : Disposable {
    fileprivate let _observer: Observer
    fileprivate let _cancel: Cancelable
    private let _disposed = AtomicInt(0)

    #if DEBUG
        private let _synchronizationTracker = SynchronizationTracker()
    #endif

    init(observer: Observer, cancel: Cancelable) {
#if TRACE_RESOURCES
        _ = Resources.incrementTotal()
#endif
        self._observer = observer
        self._cancel = cancel
    }

    final func forwardOn(_ event: Event<Observer.Element>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        if isFlagSet(self._disposed, 1) {
            return
        }
        self._observer.on(event)
    }

    final func forwarder() -> SinkForward<Observer> {
        return SinkForward(forward: self)
    }

    final var disposed: Bool {
        return isFlagSet(self._disposed, 1)
    }

    func dispose() {
        fetchOr(self._disposed, 1)
        self._cancel.dispose()
    }

    deinit {
#if TRACE_RESOURCES
       _ =  Resources.decrementTotal()
#endif
    }
}
```
整理一下，整个`subscribe`的关键就在于`AnonymousObservable.run`函数，在`run`函数中，他先创建了`AnonymousObservableSink`，并把`observer`传给了他
```swift
let sink = AnonymousObservableSink(observer: observer, cancel: cancel)
```
而`AnonymousObservableSink`是继承了`Sink`，实际上他把`observer`又传给了`Sink`  
同时注意看！`AnonymousObservableSink`遵循了`ObserverType`协议，他实现了`on`方法
```swift
func on(_ event: Event<Element>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        switch event {
        case .next:
            if load(self._isStopped) == 1 {
                return
            }
            self.forwardOn(event)
        case .error, .completed:
            if fetchOr(self._isStopped, 1) == 0 {
                self.forwardOn(event)
                self.dispose()
            }
        }
    }
```
在`on`方法里面，他实际调用的是`Sink.forwardOn`方法
```swift
final func forwardOn(_ event: Event<Observer.Element>) {
        #if DEBUG
            self._synchronizationTracker.register(synchronizationErrorMessage: .default)
            defer { self._synchronizationTracker.unregister() }
        #endif
        if isFlagSet(self._disposed, 1) {
            return
        }
        self._observer.on(event)
}
```
而`Sink.forwardOn`方法调用的是构造函数传进来的`observer`，这里接通了，也就是说现在`AnonymousObservableSink`已经传上了传进来的`observer`的衣服，调`AnonymousObservableSink`就等于调`AnonymousObserver`  
最后看，创建完`AnonymousObservableSink`后一句！
```swift
let subscription = sink.run(self)
```
而`AnonymousObservableSink.run`为：
```swift
func run(_ parent: Parent) -> Disposable {
        return parent._subscribeHandler(AnyObserver(self))
}
```
终于！在这里接通了！parent是`AnonymousObservable`，也就是序列，`_subscribeHandler`是序列里面用户传进来的创建序列的回调。`AnyObserver(self)`也就是说`AnonymousObservableSink`自己是被观察者，上面说到`AnonymousObservableSink`已经传上了`AnonymousObserver`的衣服，调他就等于调`AnonymousObserver`。因此在这里，一切都接通了！  
所以回顾一下，本质上上面做的一切操作，到最后简化下来其实就是执行一次这样的代码：
```swift
(onNext: (Any)->Void, onCompleted: ()->Void, onError: ()->Void, onDisposed: ()->Void) -> Void {
    onNext("hello")
    onCompleted()
} ( {print("recv \($0)")}, {print("complete")}, {}, {} )
```
这代码执行的时机是 `AnonymousObservableSink.run` -> `AnonymousObservable.run` -> `Producer.subscribe` -> `AnonymousObservable.subscribe` -> `observable.subscribe`

## 总结
- 序列的事件发送与观察者的代码是同时执行的
- 他们执行的时机在subscribe上