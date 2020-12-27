- Start Date: 2020-12-26
- Target Major Version: 11.x
- RFC PR: (Fill this in once RFC PR is open)
- Implementation PR: preactjs/preact#2629

# Move auto-px to compat sub-package

## Summary

Currently, Preact supports numerical values to `style` properties where the DOM
expects strings with units, for example `<div style={{ marginLeft: 20}}></div>`.
This RFC proposes moving this behavior to the `preact/compat` sub-package, such
that Preact core (the behavior when just importing `preact`) will no longer
support this. A developer will need to write `<div style={{ marginLeft: "20px"}}></div>` instead.

## Basic example

New required behavior in `preact` core. `preact/compat` remains the same.

```jsx
function App() {
  return <div style={{ marginLeft: "20px" }}></div>;
}
```

## Motivation

Related issues: preactjs/preact#2621, preactjs/preact#2607

In order for the `auto-px` feature to work properly, it needs to efficiently
list all CSS properties that do not accept a `px` unit. Keeping this list in
`preact/core` is not cheap in terms of bytes.

Further, in spirit of our "closer to the DOM" value, if a developer were using
the DOM directly, the DOM would expect that the developer would apply units as
appropriate. As such, it is reasonable for Preact to expect the same from
developers.

We've also had reports (see relevant paragraph near the top of
preactjs/preact#2621) of this feature being confusing to developers coming from
writing HTML/CSS directly.

However, because React supports this feature, we must keep it around in
`preact/compat`.

So in summary, this change helps us accomplish a couple goals:

1. Makes Preact core's API more closely match the DOM
2. Reduces the size of the core renderer
3. Maintains react compatibility

## Detailed design

This RFC would change how we diff the `style` attribute on DOM VNodes when
passed an object. When updating the style object's properties in the DOM, we
will no longer check if the style property is in the regex of
`IS_NON_DIMENSIONAL` and instead just set the value of the property directly on
the DOM as specified.

The `IS_NON_DIMENSIONAL` regex will be moved into the `preact/compat` package
and run the existing logic from core in an `options.vnode` (summarized below).

In an `options.vnode` compat hook, a DOM VNode's props object will be inspected
for a `style` prop and if the value of that prop is of type object, then run the
`IS_NON_DIMENSIONAL` regex on the name of each property of the style object and
automatically apply the `px` unit if the style object's value is of type number
and does not pass the `IS_NON_DIMENSIONAL` regex.

One possible benefit that needs to be measured is this may improve the
performance of `style` prop diffing in core by no longer having to run this
regex on every property of the `style` object. For CSS-in-JS libraries that use
inline styles but don't rely on this behavior, this could be a non-trivial
improvement (but again, I haven't measured this so I could be wrong 😅)

## Drawbacks

Some drawbacks to this change:

- It is a breaking change that requires educating developers and may require
  people to update application code
- Reduces the compatibility of Preact's core with React. There will be fewer
  React-ecosystem libraries that will just work with core and instead require
  compat
- Is slightly less ergonomic of an API to use if a developer is primarily using
  numbers to define styles inline and are having to do some numerical operations
  on the values.

## Alternatives

We could keep this behavior in core but then we'd have to continue to pay the
increasing byte cost and possible performance hit of running this code for style
props.

Another alternative could be to change the regex to something like
`IS_DIMENSIONAL` (the opposite of `IS_NON_DIMENSIONAL`). This switch could
reduce the byte cost if the opposite list of properties is smaller, but it would
still be non-zero and could still increase in the future as CSS changes.

Other libraries handle styling differently. For example, Svelte and Vue support
defining CSS in a component file so the usage of the `style` property is much
less common. However I did not investigate how they handle the style prop if
defined.

React's behavior has been previously described.

## Adoption strategy

- Add a warning to `preact/debug` sub-package
- Provide a gist that demonstrates how to add this functionality back into
  `preact` core
- Mention breaking change in 11.x release notes
- Mention breaking change in 10.x -> 11.x migration guide
- Mention breaking change in documentation
- Maybe write a eslint-plugin or js-codeshift to help migrate?

## How we teach this

Describing this change in the v11.x release notes and v11.x documentation should
suffice.

## Unresolved questions

- Could this feature be something that developers could selectively enable from
  compat? Likely this requires a change in how compat works such that people can
  pick-n-choose what levels of compat they want.
