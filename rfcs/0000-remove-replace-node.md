- Start Date: 2021-01-04
- Target Major Version: 11.x
- RFC PR: (Fill this in once RFC PR is open)
- Implementation PR: (partial start:
  [preactjs/preact#2627](https://github.com/preactjs/preact/pull/2627))

# Remove built-in support for `render`'s 3rd parameter: `replaceNode`

## Summary

Currently, apps can specify a 3rd parameter to `render`.

```jsx
render(<App />, document.body, document.body.firstChild);
```

This RFC proposes removing support for the 3rd parameter. It also proposes
workarounds for most use cases.

## Basic example

The only way to call `render` will be

```jsx
render(<App />, document.body);
```

## Background

Preact 8's `render` function accepted a third parameter that specified the child of the
container to begin hydrating or diffing with. One way to use this paramter was
if `render` was called multiple times:

```jsx
let child = render(<App update={0} />, document.body);
render(<App update={1} />, document.body, child);
```

The first original releases of Preact X (v10) removed this `replaceNode`
parameter, but a community PR added it back ([preactjs/preact#1557]).

The original implementation was focused on the use case of partial hydration. It
used the `replaceNode` parameter to specify which child in the container should
be hydrated.

Over the course of Preact X's lifetime, due to confusion with Preact 8's
`replaceNode` parameter and feature creep, the `replaceNode` parameter started
to evolve into a parameter with confusing, poorly defined behavior. It was
originally related to hydrating but became about mounting and diffing.

To understand why this behavior is confusing, let's define what each of these
terms mean:

<dl>
<dt>Hydration</dt>
<dd>

Hydration is what Preact does when there already exists DOM that the app wants
to reuse with it's initial Preact "render" or "mount". Typically this DOM was
created using a technique such as server-side rendering (SSR). By default, SSR
DOM isn't interactive so the purpose of hydration is to make it interactive.

When hydrating, Preact needs to do two things: 1) "claim" the existing DOM nodes
and attach them to the appropriate virtual nodes, and 2) attach event listeners
to the DOM nodes to make then interactive.

Typically during hydration property diffing does not happen. This behavior is to
ensure that hydration happens as fast as possible since your app won't be
interactive until it completes. The SSR's DOM is assumed to already have the
correct attributes for the initial render and so property diffing is unnecessary
for this operation.

</dd>
<dt>Mounting <a href="#footnotes"><sup>1</sup></a></dt>
<dd>

Mounting is what Preact does when it has to create new DOM nodes when rendering
an app because no DOM nodes exist. Here, Preact creates the DOM nodes your
virtual tree specifies and applies properties and attributes to it.

</dd>
<dt>Diffing</dt>
<dd>

Diffing is what Preact does when it rerenders a VNode or Component. In this
case, Preact has already mounted a virtual tree to DOM elements and now needs to
update the DOM elements to adjust to any changes. Here, Preact compares the
previously rendered virtual tree to the newly rendered virtual tree and applies
and differences to the DOM.

</dd>
</dl>

What makes the Preact X's `replaceNode` parameter confusing is that it has
evolved over time to do all three of the above at the same time. The conflation
of these ideas leads developers to expecting the wrong behavior with this
parameter, possibly worse performance, and more complex internal implementation
which increases the maintenance cost over time.

One example of the complexity of muddling these concepts together is well
demonstrated in the issues [preactjs/preact#2496], [preactjs/preact#2500],
[preactjs/preact#2791].

These issues currently expose a bug with diffing while using `replaceNode`. For
all these issues, the second call to `render` triggers a re-mount of a VNode
(either through a new key or new Component instance). This means there is a new
VNode to be mounted (the one with the new key) and an old VNode to be unmounted
(the one with the old key). When the new VNode to be mounted is diffed, because
the `replaceNode` parameter is present, Preact looks at existing DOM nodes to
"claim". This behavior is to support the "hydration" use case of `replaceNode`.
In this case, the new VNode "claims" the existing DOM child of `replaceNode`.
However, this DOM element is the same DOM element that was already "claimed" in
the initial call to `render` by the old VNode that is be unmounted. So once the
old VNode is unmounted, the DOM element that is claimed by both the old VNode
and the new VNode is removed from the document.

Trying to `hydrate` and `diff` at the same time can lead to subtle bugs such as
this one. Avoiding these kinds of difficult to catch and debug issues is a
reason to drop support for the `replaceNode` parameter.

## Motivation

The primary motivation for removing this feature is to reduce confusion for
developers trying to upgrade to Preact and to reduce the complexity of Preact's
implementation.

By removing the parameter, we can rewrite our internals to fit the model of
"hydration", "mounting" and "diffing" described above which would reduce size,
runtime, and maintenance cost of Preact.

The "Research" section at the bottom outlines PRs and issues related to
replaceNode and the difficulty in maintaining it.

## Detailed design

Here we'll outline the various existing use cases for `replaceNode` and suggest
workarounds for each.

### Use cases

**TODO: Go through the various use cases and demonstrate a workaround**

<dl>
<dt>Partial hydration</dt>
<dd>

Specify which child of a parent to begin hydrating with. Assumes fully SSR'ed
app. From initial Preact X implementation.

Associated issues:

- [preactjs/preact#1557]

</dd>

<dt>Specify child to begin **rendering** with and **diff**</dt>
<dd>

Specify which child of a parent to being **rendering** with. Expectation is
full, corrective property diffing.

Currently has bugs when using `replaceNode` (see "Full diffing with
`replaceNode`" section below).

Closed:

- [preactjs/preact#1645] Unmount with replacing components
- [preactjs/preact#1665] Wants to diff two different components in one container
- [preactjs/preact#1722] Creates marker element Preact should mount into
- [preactjs/preact#1783] Wants property diffing on `replaceNode`

Still open:

- [preactjs/preact#2004] Various cases generally involving diffing
- [preactjs/preact#2496] Doing a full diff while unmounting element with key
- [preactjs/preact#2500] Doing a full diff while unmounting matching type
- [preactjs/preact#2791] Doing a full diff while unmounting element with key

</dd>
</dl>

## Drawbacks

> Why should we _not_ do this? Please consider:
>
> - implementation cost, both in term of code size, performance, and complexity
> - whether the proposed feature can be implemented in user space
> - the impact on teaching people Preact
> - integration of this feature with other existing and planned features
> - cost of migrating existing Preact applications (is it a breaking change?)
>
> There are tradeoffs to choosing any path. Attempt to identify them here.

## Alternatives

**TODO Mention alternative could be to fix remaining bugs with `replaceNode` but
how that introduces more complexity and size we'd rather spend on other features
(e.g. progressive hydration).**

**TODO: mention workaround in [preactjs/preact#2168]**

By default, using `render` without `replaceNode` will always append to the
parent. This behavior can lead to extra wrapping divs. The former `replaceNode`
becomes the parent of the Preact app which may also render a top-level `div`.

Some workarounds:

- Use a ["parent root fragment"](https://gist.github.com/developit/7c1b983dbd2cb68e6cefd367dfcf0ca1)
  to pass to the `parent` parameter of `render`. [Issue where developer claims it works for them](https://github.com/preactjs/preact/issues/2791)
- To avoid a wrapper div, render a Preact Fragment into the wrapper (or formerly
  `replaceNode`). Works well if wrapper div doesn't change based on component
  state.

  ```jsx
  var wrapper = document.createElement("div");
  document.body.appendChild(wrapper);
  render(
    <Fragment>
      <A />
      <B />
    </Fragment>,
    wrapper
  );
  ```

> What other designs have been considered? What is the impact of not doing this?
>
> This section could also include prior art, that is, how other frameworks in
> the same domain have solved this problem similarly or differently.

## Adoption strategy

> If we implement this proposal, how will existing Preact developers adopt it?
> Is this a breaking change? Can we write a codemod? Can we provide a runtime
> adapter library for the original API it replaces? How will this affect other
> projects in the Preact ecosystem? Should we coordinate with other projects or
> libraries?

## How we teach this

> How should this feature be taught to existing Preact developers?
>
> What names and terminology work best for these concepts and why? How is this
> idea best presented? As a continuation of existing Preact patterns, or as a
> wholly new one?
>
> Would the acceptance of this proposal mean the Preact documentation must be
> re-organized or altered? Does it change how Preact is taught to new developers
> at any level?

## Unresolved questions

- Unrelated, but we should probably re-clarify to developers that our `render`
  function doesn't remove the contents of the `parent` DOM unlike React.
  Developers will need to do this themselves before calling `render` if that
  behavior is desired. (e.g. [preactjs/preact#2522])

## Footnotes

<p><sup>1</sup> Technically you could say hydration _mounts_ components to existing DOM elements while this operation mounts components to newly created DOM elements (hence the lifecycle `componentDidMount`). However for this document I'm breaking these apart and calling mounting with existing DOM elements "hydration" and mounting with new DOM elements "mounting"
</p>

## Research

<details>
<summary>Initial research into various issues and PRs related to <code>replaceNode</code></summary>

In summary, seems like `replaceNode` in Preact X was originally meant for
hydration but due to feature creep and confusion with Preact 8's `replaceNode`
param, it's behavior became partially mixed with rendering and diffing.

### PRs & Issues

Related PRs and their associated issues:

<dl>

<dt><a href="https://github.com/preactjs/preact/pull/1557">#1557</a> (feat) - replaceNode parameter</dt>
<dd>

Initial PR to add `replaceNode` parameter to Preact X. From reading the
gist and medium article below, the main purpose of the `replaceNode` parameter
was to specify which DOM child of the in the parent DOM element Preact should
begin hydrating with.

In summary, specify which child of `parent` to **hydrate**. This behavior avoids
having to wrap the preact application in extra parent div on the server and
client.

Associated links:

- [Gist describing original use case for adding it back](https://gist.github.com/LukasBombach/884319d5430a3fb85f3b4385d7a31c89)
- [Medium article explaining partial hydration approach](https://medium.com/@luke_schmuke/how-we-achieved-the-best-web-performance-with-partial-hydration-20fab9c808d5)

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/1647">#1647</a> (fix) - should call unmount when replacing components</dt>
<dd>

Subsequent calls to `render` with `replaceNode` parameter should
properly unmount the top-level component if it changes. For example, calling
`render(<A />, parent)`, then `render(<B />, parent, child)` should call unmount
lifecycles on `A`. This behavior already exists when calling `render` without a
`replaceNode` param so this change just brought `replaceNode` to parity to
normal render. The next PR (preactjs/preact#1723) fixes this behavior when
calling `render(<A />, parent, child)` then `render(<B />, parent, child)`.

In summary, specify which child of `parent` to **diff & render** into. Also
avoids a wrapping div just on the client.

Associated issues:

- [preactjs/preact#1645]: componentWillUnmount does not fire when using Preact.render() with replaceNode]()

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/1723">#1723</a> (fix) - replaceNode should unmount when diffing twice with replaceNode</dt>
<dd>

This PR stores the rendered virtual tree on the `replaceNode` element instead of
the parent DOM element which allows for the behavior mentioned in
preactjs/preact#1665 and fixes preactjs/preact#1722 by ensuring calling `render`
with a `replaceNode` will always try to diff the new virtual tree with the
previous virtual tree of that `replaceNode`.

[preactjs/preact#1665] appears to be **rendering** apps into children of a parent
again to avoid a wrapper div on the client.

[preactjs/preact#1722] appears to be creating a marker element to specify exactly
what child Preact should **render** an app.

Associated issues:

- [preactjs/preact#1665]: Does `preact.render` support multiple components
  within a single parent node?
- [preactjs/preact#1722]: componentWillUnmount not called

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/1786">#1786</a> (feat) - diff props when replacing a node and don't diff when hydrating</dt>
<dd>

In this issue and PR, we special case the `replaceNode` parameter to apply props
to existing DOM.

In summary, this issue and PR make `replaceNode` close to **rendering** and not
hydration since props are applied.

This use case mixes rendering and hydration. Typically hydration is used to
"claim" existing DOM nodes and place them into a new virtual tree. However
hydration doesn't apply attributes (ideally it only adds event listeners). But
this use case is really trying to "render" a new app into a container - it just
needs more control over where that container lives (i.e. a specific child of a
parent) and so just the first dom is created and needs to be "claimed" while all
of its children are created from scratch.

Follow up PRs:

- [preactjs/preact#1802]: simple follow up to use a special internal flag for
  hydration

Associated issues:

- [preactjs/preact#1783]: Render bug with replaceNode parameter after update
  from 10.0.0-beta.3 to 10.0.0-rc.0

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/1900">#1900</a> (fix) - replacing a node</dt>
<dd>

The issue here was caused by first calling `render` with two args then `render`
with 3 args (including a `replaceNode`). This behavior caused the diff engine to
first create DOM in the first `render` and then in the second call to `render`
attempt to "re-claim" the same dom it had previously already created. This
second attempt to re-claim is unnecessary because the DOM created in the first
render is already tracked by the store virtual tree. There is no need to
re-claim it.

Note, normal hydration didn't invoke this code path (it hit a `dom==null` code
path that was correct) and so was specific to calling render with `replaceNode`
after a render or hydration.

Note this PR is superseded by [preactjs/preact#2356] in case you want to follow
along how the code has changed.

Follow up PRs:

- [preactjs/preact#1970]: Make this behavior work for calling `render` with
  `replaceNode` twice

Associated issues:

- [preactjs/preact#1896]: preact.render third argument causes weird behavior

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2195">#2195</a> Added unit tests to check proper component unmounting</dt>
<dd>

This PR adds a test that came out of the discussion in the associated issue
below. The original issue is about a Preact 8 ability to unmount unrelated
"islands" of Preact components from a non-Preact parent root. Take the following
example:

```jsx
<body>
  <header></header>
  <div id="main">
    <article>
      <div id="article">
        {/* Mounted using render(<Article />, article) */}
        <Article1 />
      </div>
    </article>
    <div id="aside">
      {/* Mounted using render(<Aside />, aside) */}
      <Aside />
    </div>
  </div>
</body>
```

If a client-side navigation were to happen and the app wanted to unmount all
components under `main`, using Preact 8 a developer could call something like
`render(<div />, document.body, main)` without having to know or care about
which DOM under `main` was rendered by Preact components. Preact would climb all
children under `main` and unmount all children.

This capability is no longer supported in Preact X since it no longer diffs
against the DOM but instead maintains an in-memory copy of the virtual tree and
diffs against that. This change implies that Preact now roots its render at the
container element passed in (i.e. stores the previous virtual tree on the
container). As such, subsequent rerenders must be called with the some container
element so that the old virtual tree can be compared to the new virtual tree.

Associated Issues

- [preactjs/preact#2168]: Unmount not called for nested children on render (since preact X)

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2274">#2274</a> avoid removing existing dom nodes on subsequent replaceNode calls</dt>
<dd>

This PR is similar to #1900 but applies to DOM nodes as well not just text
nodes. When calling `render` with a `replaceNode` after first calling `render`,
we didn't mark DOM nodes as used and so some would get removed as excess. As
with #1900, `normal` hydrate hit a different code path which didn't invoke this
bug.

Note this PR is superseded by [preactjs/preact#2356] in case you want to follow
along how the code has changed.

Associated Issues

- [preactjs/preact#2260]: App breaks after root update if rendered with replaceNode
- [preactjs/preact#2264]: Preact 10 rendering into document.body clobbers existing DOM nodes

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2210">#2210</a> [DRAFT] Handle subtree replaced via replaceNode</dt>
<dd>

This PR fixes an issue where calling `render` with `replaceNode` doesn't handle
the case where the result of `render` changes the DOM element of `replaceNode`.

Associated Issues:

- [#2004 Render / replaceNode unexpected behavior?](https://github.com/preactjs/preact/issues/2004)

</dd>

</dl>

Some prominent issues:

<dl>

<dt>Attribute merging</dt>
<dd>

Here are some issues that expect attribute **merging** instead of diffing.
Existing DOM attributes should be left alone while vnode attributes should be
added.

- [preactjs/preact#2022]
- [preactjs/preact#2449]

</dd>

<dt>Full diffing with `replaceNode`</dt>
<dd>

In a couple of issues the developer expected Preact to do a proper diff when
calling `render` with the `replaceNode`. This expectation further "muddles" the
meaning of `replaceNode`: is it used for hydration and claiming DOM nodes
(mostly how it is coded today) or for diffing (the feature creep of
`replaceNode`).

These issues currently expose a bug with diffing while using `replaceNode`. For
all these issues, the second render triggers a re-mount of a VNode (either
through a new key or new type). This means there is a new VNode to be mounted
(the one with the new key) and an old VNode to be unmounted (the one with the
old key). When the new VNode to be mounted is diffed, because the `replaceNode`
parameter is present, `excessDomChildren` is not null. This causes the new VNode
to "claim" the existing DOM child of `replaceNode`. However, this DOM is the
same DOM that was already "claimed" in the initial render by the old VNode that
is be unmounted. So once the old VNode is unmounted, the DOM is removed from the
document.

The above paragraph demonstrates the complexity of the `replaceNode`. Originally
it was intended to be used only for hydration, but due to feature creep and
Preact 8 migration confusion, there is an expectation that it also does full
rendering. In the presence of the `replaceNode` param, how do we know deep in a
tree if a DOM child should be claimed or if a sub tree should be rebuilt.
Hydration always claims, normal rendering always rebuilds.

- [preactjs/preact#2496] (unmounting element with key)
- [preactjs/preact#2500] (unmounting matching type)
- [preactjs/preact#2791] (unmounting element with key)

</dd>

</dl>

**TODO: Investigate:**

- [#2004 Render / replaceNode unexpected behavior?](https://github.com/preactjs/preact/issues/2004)

</details>

[preactjs/preact#1645]: https://github.com/preactjs/preact/issues/1645
[preactjs/preact#1665]: https://github.com/preactjs/preact/issues/1665
[preactjs/preact#1722]: https://github.com/preactjs/preact/issues/1722
[preactjs/preact#1783]: https://github.com/preactjs/preact/issues/1783
[preactjs/preact#1802]: https://github.com/preactjs/preact/pull/1802
[preactjs/preact#1896]: https://github.com/preactjs/preact/issues/1896
[preactjs/preact#2356]: https://github.com/preactjs/preact/pull/2356
[preactjs/preact#1970]: https://github.com/preactjs/preact/pull/1970
[preactjs/preact#2004]: https://github.com/preactjs/preact/issues/2004
[preactjs/preact#2022]: https://github.com/preactjs/preact/issues/2022
[preactjs/preact#2449]: https://github.com/preactjs/preact/issues/2449
[preactjs/preact#2168]: https://github.com/preactjs/preact/issues/2168
[preactjs/preact#2260]: https://github.com/preactjs/preact/issues/2260
[preactjs/preact#2264]: https://github.com/preactjs/preact/issues/2264
[preactjs/preact#2496]: https://github.com/preactjs/preact/issues/2496
[preactjs/preact#2500]: https://github.com/preactjs/preact/issues/2500
[preactjs/preact#2522]: https://github.com/preactjs/preact/issues/2522
[preactjs/preact#2791]: https://github.com/preactjs/preact/issues/2791
