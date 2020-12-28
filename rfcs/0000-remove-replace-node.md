- Start Date: 2020-27-12
- Target Major Version: 11.x
- RFC PR: (Fill this in once RFC PR is open)
- Implementation PR: (partial start:
  [preactjs/preact#2627](https://github.com/preactjs/preact/pull/2627))

# (RFC title goes here)

## Summary

> Brief explanation of the feature.

## Basic example

> If the proposal involves a new or changed API, include a basic code example.
> Omit this section if it's not applicable.

## Research

### Use cases

In summary, seems like `replaceNode` in Preact X was originally meant for
hydration but due to feature creep and confusion with Preact 8's `replaceNode`
param, it's behavior became partially mixed with rendering and diffing.

<dl>
<dt>Partial hydration</dt>
<dd>

Specify which child of a parent to begin hydrating with. Assumes fully SSR'ed
app. From initial Preact X implementation.

</dd>

<dt>Specify child to begin **rendering** with and **diff**</dt>
<dd>

Specify which child of a parent to being **rendering** with. Expectation is
full, corrective property diffing.

Currently has bugs when using `replaceNode` (see "Full diffing with
`replaceNode`" section below).

</dd>

<dt>Specify child to begin **rendering** with and **merge**</dt>
<dd>

Specify which child of a parent to being **rendering** with. Expectation is
props should be merged with existing DOM attributes.

</dd>
</dl>

### PRs & Issues

Related PRs and their associated issues:

<dl>

<dt><a href="https://github.com/preactjs/preact/pull/1557">#1557 (feat) - replaceNode parameter</a></dt>
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

<dt>
<a href="https://github.com/preactjs/preact/pull/1647">#1647 (fix) - should call unmount when replacing components]()</a>
</dt>
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

<dt><a href="https://github.com/preactjs/preact/pull/1723">#1723 (fix) - replaceNode should unmount when diffing twice with replaceNode</a></dt>
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

<dt><a href="https://github.com/preactjs/preact/pull/1786">#1786 (feat) - diff props when replacing a node and don't diff when hydrating</a></dt>
<dd>

In this issue and PR, we special case the `replaceNode` parameter to apply props
to existing DOM.

In summary, this issue and PR make `replaceNode` close to **rendering** and not
hydration since props are applied.

**TODO: LOOK HERE** - this use case mixes rendering and hydration. Typically
hydration is used to "claim" existing DOM nodes and place them into a new
virtual tree. However hydration doesn't apply attributes (ideally it only adds
event listeners). But this use case is really trying to "render" a new app into
a container - it just needs more control over where that container lives (i.e. a
specific child of a parent) and so just the first dom is created and needs to be
"claimed" while all of its children are created from scratch. Need to think more
about how this accommodate this scenario.

Follow up PRs:

- [preactjs/preact#1802]: simple follow up to use a special internal flag for
  hydration

Associated issues:

- [preactjs/preact#1783]: Render bug with replaceNode parameter after update
  from 10.0.0-beta.3 to 10.0.0-rc.0

</dd>

<dt>[#1900 (fix) - replacing a node](https://github.com/preactjs/preact/pull/1900)</dt>
<dd>

The issue here was caused by first calling `render` with two args then `render`
with 3 args (including a `replaceNode`). This behavior caused the diff engine to
first create DOM in the first `render` and then in the second call to `render`
attempt to "re-claim" the same dom it had previously already created. This
second attempt to re-claim is unnecessary because the DOM created in the first
render is already tracked by the store virtual tree. There is no need to
re-claim it.

However this calling pattern exposed a bug where we weren't properly claiming
text nodes when `excessDomChildren` wasn't null, causing them to be removed.

Note this PR is superseded by [preactjs/preact#2356] in case you want to follow
along how the code has changed.

**TODO: Investigate if this could've occurred doing normal hydration or if it
required the double `render` call pattern to reproduce**

Follow up PRs:

- [preactjs/preact#1970]: Make this behavior work for calling `render` with
  `replaceNode` twice

Associated issues:

- [preactjs/preact#1896]: preact.render third argument causes weird behavior

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2195">#2195 Added unit tests to check proper component unmounting</a></dt>
<dd>

**TODO**

Associated Issues

- [preactjs/preact#2168]: Unmount not called for nested children on render (since preact X)

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2274">#2274 avoid removing existing dom nodes on subsequent replaceNode calls</a></dt>
<dd>

**TODO**

Associated Issues

- [preactjs/preact#2260]: App breaks after root update if rendered with replaceNode
- [preactjs/preact#2264]: Preact 10 rendering into document.body clobbers existing DOM nodes

</dd>

<dt><a href="https://github.com/preactjs/preact/pull/2210">#2210 [DRAFT] Handle subtree replaced via replaceNode</a></dt>
<dd>

**TODO**

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

<!-- When calling `render` twice with the `replaceNode` parameter, there are two
competing sources of truth: the old virtual tree (with it's associated DOM
nodes) and the DOM tree of `replaceNode`. Which do we use? Currently passing
in `replaceNode` always favors claiming `excessDomChildren`. This behavior was
unexpected by the developer in [preactjs/preact#2791] who was using
`replaceNode` with VNodes with keys.

Fundamentally, we have different heuristics for matching the new VNode tree to
the old virtual tree (this uses types and keys) vs when trying to claim
existing DOM nodes (only look at type). -->

</dd>

</dl>

**TODO: Investigate:**

- [#2004 Render / replaceNode unexpected behavior?](https://github.com/preactjs/preact/issues/2004)

## Motivation

Much of the confusion can likely be attributed to the difference between Preact
X and Preact 8 where Preact 8 required the 3-arg to do diffing and Preact X did
not. A lesson learned here to perhaps better understand developers use cases and
educate developers on changes.

Examples of confusion:

- Should not be using both 2-arg render and 3-arg render together:
  - https://github.com/preactjs/preact/issues/1896
  - https://github.com/preactjs/preact/issues/2008
- Expected `replaceNode` to behave like React and be the DOM node that your app
  literally "replaces":
  - https://github.com/preactjs/preact/issues/2522

> Why are we doing this? What use cases does it support? What is the expected
> outcome?
>
> Please focus on explaining the motivation so that if this RFC is not accepted,
> the motivation could be used to develop alternative solutions. In other words,
> enumerate the constraints you are trying to solve without coupling them too
> closely to the solution you have in mind.
>
> Please provide specific examples. If you say "this would be more flexible"
> then give an example of something that becomes easier. If you say "this would
> be make it easier to do X" then give an example of what that looks like today
> and what's hard about it.
>
> Don't assume that others recognize the problem is one that needs to be solved.
> Is there some concrete issue you cannot accomplish without this? What does it
> look like to accomplish some set of goals today and how could that be
> improved? Are there any workarounds that are necessary today? Are there open
> issues on Github where people would be helped by this?

## Detailed design

> This is the bulk of the RFC. Explain the design in enough detail for somebody
> familiar with Preact to understand, and for somebody familiar with the
> implementation to implement. This should get into specifics and corner-cases,
> and include examples of how the feature is used. Any new terminology should be
> defined here.

To consider: Should our hydration bailout into normal rendering + appending if
it can not find the proper matching element to hydrate? Doing this would support
the `replaceNode` scenario of using a `div` marker (see
[preactjs/preact#1722](https://github.com/preactjs/preact/issues/1722)) to
specify where to begin **rendering** an app. So while the intention is to
_render_ (the entire app wasn't SSR'ed so hydration doesn't really make sense)
we'd still use hydration to capture the existing DOM marker (formerly the
`replaceNode`) and then bailout to normal rendering once we realize their are no
other children in the parent. I'd imagine hydration does not search for matching
DOM node but just uses the DOM nodes in place in order they appear as long as
they exist.

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

> Optional, but suggested for first drafts. What parts of the design are still
> TBD?

[preactjs/preact#1645]: https://github.com/preactjs/preact/issues/1645
[preactjs/preact#1665]: https://github.com/preactjs/preact/issues/1665
[preactjs/preact#1722]: https://github.com/preactjs/preact/issues/1722
[preactjs/preact#1783]: https://github.com/preactjs/preact/issues/1783
[preactjs/preact#1802]: https://github.com/preactjs/preact/pull/1802
[preactjs/preact#1896]: https://github.com/preactjs/preact/issues/1896
[preactjs/preact#2356]: https://github.com/preactjs/preact/pull/2356
[preactjs/preact#1970]: https://github.com/preactjs/preact/pull/1970
[preactjs/preact#2022]: https://github.com/preactjs/preact/issues/2022
[preactjs/preact#2449]: https://github.com/preactjs/preact/issues/2449
[preactjs/preact#2168]: https://github.com/preactjs/preact/issues/2168
[preactjs/preact#2260]: https://github.com/preactjs/preact/issues/2260
[preactjs/preact#2264]: https://github.com/preactjs/preact/issues/2264
[preactjs/preact#2496]: https://github.com/preactjs/preact/issues/2496
[preactjs/preact#2500]: https://github.com/preactjs/preact/issues/2500
[preactjs/preact#2522]: https://github.com/preactjs/preact/issues/2522
[preactjs/preact#2791]: https://github.com/preactjs/preact/issues/2791
