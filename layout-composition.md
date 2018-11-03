[Additional Docs and Tips (Mithril 2.x)](./readme.md)

# Composing Layouts
## (Or, "How do I create master templates?")

If you've used other templating systems you're probably familiar with, and have relied on, the ability to create a "master" layout that contains common elements across a series of pages without having to repeat those manually. You can accomplish the same thing rather easily with Mithril by making use of the `Vnode.children` property.

Suppose we have some pages that will all have a Header, a side nav and a footer. We can create a "Layout" component like this:

```javascript
const Layout = {
    view: function() {
        return m(".layout",
            m(Header),
            m(SideNav),
            m(Footer)
        )
    }
}
```

Now we want to put the content that's unique for each page in-between the SideNav and Footer components. That's as easy as doing:

```javascript
const Layout = {
    view: function(vnode) {
        return m(".layout",
            m(Header),
            m(SideNav),
            vnode.children,
            m(Footer)
        )
    }
}
```

Now we can set up some routes to render each page:

```javascript
m.route(document.body, "/", {
    "/home": {
        render: function() {
            return m(Layout, m(HomePage))
        }
    },
    "/news": {
        render: function() {
            return m(Layout, m(NewsPage))
        }
    },
    "/about": {
        render: function() {
            return m(Layout, m(AboutPage))
        }
    }
})
```

## More complex layouts

### Send vnodes via attrs

Maybe you'd like to inject more than just one single tree of content into a layout. Let's say you want to add an "Intro" section between the Header and the SideNav in addition to the main content. Instead of sending one piece of content as `children` we can send two pieces via `attrs`:

```javascript
const Layout = {
    view: function(vnode) {
        return m(".layout",
            m(Header),
            vnode.attrs.introContent,
            m(SideNav),
            vnode.attrs.mainContent,
            m(Footer)
        )
    }
}
```

And we coud use it like so:

```javascript
m(Layout, {
    introContent: m("p", "..."),
    mainContent: m("main", "...")
})
```

### Send view functions via attrs

We could just as easily send functions to render content if we prefer:

```javascript
const Layout = {
    view: function(vnode) {
        return m(".layout",
            m(Header),
            vnode.attrs.introView(),
            m(SideNav),
            vnode.attrs.mainView(),
            m(Footer)
        )
    }
}
```

Used like:

```javascript
m(Layout, {
    introView: function() {
        return m("p", "...")
    },
    mainView: function() {
        return m("main", "...")
    }
})
```
