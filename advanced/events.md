---
title: Application monitoring
caption: Handling application-level events
category: advanced
permalink: /advanced/events.html
keywords: >-
    EventDefinition events monitoring monitors
    ApplicationStarting ApplicationStarted ApplicationStopPreparing ApplicationStopping ApplicationStopped
    subscribing unsubscribing raising raise events dispatching
---

At server-side, in addition to handling requests, Ktor exposes a mechanism to produce and consume
global events.

For example, when the application has started, it is starting, or has stopped, an event is produced and you can
subscribe or unsubscribe to those events to execute code when that happens.
To do that, associated to the application environment, there is a `monitor: ApplicationEvents` instance
acting as an event dispatcher.

The `ApplicationEvents` dispatches typed `EventDefinition<T>` along an object `T`.

You can get the monitor along the application instance by executing `application.environment.monitor`.

### API

The simplified API for the `monitor: ApplicationEvents` looks like this:

```kotlin
class ApplicationEvents {
    fun <T> subscribe(definition: EventDefinition<T>, handler: EventHandler<T>): DisposableHandle
    fun <T> unsubscribe(definition: EventDefinition<T>, handler: EventHandler<T>)
    fun <T> raise(definition: EventDefinition<T>, value: T)
}

class EventDefinition<T>

typealias EventHandler<T> = (T) -> Unit

interface DisposableHandle {
    fun dispose()
}
```

### Predefined EventDefinitions

Ktor provides some predefined events that are dispatched by the engine:

```kotlin
val ApplicationStarting: EventDefinition<Application>
val ApplicationStarted: EventDefinition<Application>
val ApplicationStopPreparing: EventDefinition<ApplicationEnvironment>
val ApplicationStopping: EventDefinition<Application>
val ApplicationStopped: EventDefinition<Application>
```

### Subscribing to events and raising them

You can subscribe to events by calling the `subscribe` method from the monitor. The subscribe method returns a
`DisposableHandle` that you can call to cancel the subscription. Additionally you can call the `unsubscribe`
method with the same method handle to cancel the subscription.

Using the disposable:

```kotlin
val disposable = application.environment.monitor.subscribe(ApplicationStarting) { application: Application ->
    // Handle the event using the application as subject
}
disposable.dispose() // Cancels the subscription
```

Using a lambda stored in a property:

```kotlin
val starting: (Application) -> Unit = { log("Application starting: $it") }

application.environment.monitor.subscribe(ApplicationStarting, starting) // subscribe
application.environment.monitor.unsubscribe(ApplicationStarting, starting) // unsubscribe
```

Using a method reference:

```kotlin
fun starting(application: Application) { log("Application starting: $it") }

application.environment.monitor.subscribe(ApplicationStarting, ::starting) // subscribe
application.environment.monitor.unsubscribe(ApplicationStarting, ::starting) // unsubscribe
```

If you want to create custom events and dispatching/raising them:

```kotlin
class MySubject
val MyEventDefinition = EventDefinition<MySubject>()
monitor.raise(MyEventDefinition, MySubject())
```

### Examples

You can check the [`CallLogging` feature source code](https://github.com/ktorio/ktor/blob/45d7487b82b9dfc281a8c56c1dd3989ccf67bb5d/ktor-server/ktor-server-core/src/io/ktor/features/CallLogging.kt) that includes code subscribing to events from the application.
