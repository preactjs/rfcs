- Start Date: 2020-12-27
- Target Major Version: 11.x
- RFC PR: [preactjs/rfcs#6](https://github.com/preactjs/rfcs/pull/6)
- Implementation PR: [preactjs/preact#2850](https://github.com/preactjs/preact/pull/2850)

# Forward ref by default

## Summary

Currently Preact will give you a reference to the virtual dom-node when passing ref
to a functional component. This seems kind off counter-intuitive since there's not much
a user can do with a reference to a vnode. This rfc suggests forwarding the ref into props
for functional components so it's easier to attach to a dom-node. Currently there is a way
to achieve this in the `preact/compat` package under the name of `forwardRef`, this is a higher
order function that allows you to achieve this behavior. Problem with this function is that
now all these components are marked as side-effectful since we invoke a function by default.
This can grow bundle-size in certain bundlers.

## Basic example

Let's look at how this works in practice.

In our current situation:

```jsx
import { useRef } from 'preact/hooks';

const Component = () => {
  const ref = useRef()
  return (
    <div ref={ref}>Foo</div>
  )
}
```

This will give us a reference to the HTML-node representing this `div`. We don't want to change
anything to this scenario.

```jsx
import { forwardRef } from 'preact/compat';
import { useRef } from 'preact/hooks';

const Child = forwardRef((props, ref) => {
  return (
    <div ref={ref}>Foo</div>
  )
});

const Component = () => {
  const ref = useRef()
  return (
    <Child ref={ref} />
  )
}
```

Without the `forwardRef` our `ref` constant would represent the `vnode` of type `Child`, with the
`forwardRef` we'll have the `div` dom-node since we are forwarding the ref.

Now with the newly implemented situation:

```jsx
import { useRef } from 'preact/hooks';

const Child = (props) => {
  return (
    <div ref={props.ref}>Foo</div>
  )
};

const Component = () => {
  const ref = useRef()
  return (
    <Child ref={ref} />
  )
}
```

We'll have ref forwarded into props which eliminates the need for `forwardRef`.

## Motivation

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

## Drawbacks

- It's a breaking change and could introduce some issues with pure Preact
libraries relying on this behavior.
- We'll have to be backwards compatible for React environments, this means that
we can't have this behavior in compat.

## Alternatives

Not doing this keeps our current setup around which for me feels like it's kinda
counter-intuitive.

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

> Optional, but suggested for first drafts. What parts of the design are still
> TBD?
