#[Additional Docs and Tips (Mithril 2.x)](./readme.md)

# Parent-Child Component Communication

Sending data one-way to a child component is straightforward:

``` javascript
const Parent = {
    view: function() {
        return m(Child, {
            foo: 1
        })
    }
}

const Child = {
    view: function(vnode) {
        return m(".foo", vnode.attrs.foo)
    }
}
```

What if you want to allow the Child component to affect the value `foo`? A simple solution is to provide a callback function to the child via `attrs` so that it can alert the parent whenever the value changes. In this example, we're using a closure component to hold some state in the parent while the child component remains stateless.

```javascript
function App() {
    var days = 1

    return {
        view: function() {
            m(DaysInput, {
                days: days,
                onChanged: function(value) {
                    days = value
                }
            })
        }
    }
}

const DaysInput = {
    view(vnode) {
        return m("input", {
            type: "number",
            value: vnode.attrs.days,
            oninput: m.withAttr("value", function(value) {
                vnode.attrs.onChanged(Number(value))
            })
        })
    }
}
```
