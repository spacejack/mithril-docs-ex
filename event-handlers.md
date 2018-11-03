[Additional Docs and Tips (Mithril 2.x)](./readme.md)

# Working with Event Handlers

The simplest way to use events in Mithril is to write them inline:

```javascript
m("button",
    {
        onclick: function(e) {
            // Do something
        }
    },
    "Click Me"
)
```

In some situations, it may be undesirable to re-create that function every render. Or you might not want to write event handler logic inline in the view. In which case you can declare that function external to the view and use it like so:

```javascript
function handleClick() {
    // Do something
}

// ...

m("button",
    {
        onclick: handleClick
    }
    "Click Me"
)
```

However you might want the `handleClick` function to have access to a component's state. An easy way to provide access to component state and avoid re-creating the event handler every render is with a closure component:

```javascript
function MyComponent() {
    var count = 0

    function increment() {
        count += 1
    }

    return {
        view: function() {
            return m(".app",
                m("p", "Count: " + count),
                m('button',
                    {
                        onclick: increment
                    },
                    "Increment"
                )
            )
        }
    }
}
```

If you prefer to use a POJO or class component to hold state, then you can take advantage of the fact that Mithril also accepts [EventListener](https://developer.mozilla.org/en-US/docs/Web/API/EventListener) objects as well as functions for event handlers:

```javascript
class MyComponent {
    constructor() {
        this.count = 0
    }

    handleEvent(e) {
        this.count += 1
    }

    view() {
        return m(".app",
            m("p", "Count: " + count),
            m('button',
                {
                    onclick: this
                },
                "Increment"
            )
        )
    }
}
```

Any object with a `handleEvent` method can be used as an `EventListener`. In the above example, our class has this method so we can use `this` directly to handle the event. However in this case, if we want to handle more events in this view, then handleEvent will need to be able to distinguish between them.

```javascript
class MyComponent {
    constructor() {
        this.count = 0
    }

    handleEvent(e) {
        const action = e.currentTarget.dataset.action
        if (action === "increment") {
            this.count += 1
        } else if (action === "decrement") {
            this.count -= 1
        } else {
            // other cases...
        }
    }

    view() {
        return [
            m('button',
                {
                    "data-action": "increment"
                    onclick: this
                },
                "Increment"
            ),
            m('button',
                {
                    "data-action": "decrement",
                    onclick: this
                },
                "Decrement"
            )
        ]
    }
}
```

Invoking class or POJO methods from event handler functions has some caveats if you want to use `this`. The following example will *not* work:

```javascript
class MyBrokenComponent {
    constructor() {
        this.count = 0
    }

    increment() {
        // `this` will point to the event, not our component instance
        this.count += 1
    }

    view() {
        return m("button",
            {
                // not bound to `this` when invoked
                onclick: this.increment
            },
            "Increment"
        )
    }
}
```

Given that we're using a `class` (and therefore ES2015 or later) we can wrap the call to `increment` in an arrow function:

```javascript
class MyComponent {
    constructor() {
        this.count = 0
    }

    increment() {
        this.count += 1
    }

    view() {
        return m("button",
            {
                onclick: () => {
                    this.increment()
                }
            },
            "Increment"
        )
    }
}
```

When using a stateful POJO component, you're better off using `vnode.state` rather than `this`, especially in cases where you're using ES5:

```javascript
var MyComponent = {
    oninit: function(vnode) {
        vnode.state.count = 0
    },

    increment: function(vnode) {
        vnode.state.count += 1
    },

    view: function(vnode) {
        return m("button",
            {
                onclick: function() {
                    vnode.state.increment(vnode)
                }
            }
        )
    }
}
```
