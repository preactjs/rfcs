- Start Date: 2020/12/26
- Target Major Version: 11.x
- RFC PR: (Fill this in once RFC PR is open)
- Implementation PR: [preactjs/preact#2627](https://github.com/preactjs/preact/pull/2627)

# Replace top-level `render` and `hydrate` with new `createRoot` API

## Summary

Currently, the `preact` package exports two functions `render` and `hydrate`.
This RFC proposes a single new API to replace these two functions: `createRoot`.
`createRoot` would accept a single parameter, `parent`, that is the parent DOM
element where Preact should place its children under. `createRoot` returns an
object with two methods: `render` and `hydrate`. Each of these functions take
the same parameters of the previous `render` and `hydrate` functions minus the
`parent` parameter.

## Basic example

```jsx
import { createRoot } from "preact";
import App from "./App.js";

const parent = document.getElementById("app");

// To hydrate
createRoot(parent).hydrate(<App />);

// To render
createRoot(parent).render(<App />);
```

## Motivation

Related issues: [preactjs/preact#2621](https://github.com/preactjs/preact/issues/2621)

**TODO: Mention how removing replaceNode is a separate RFC.**

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

Formal TypeScript interface:

```ts
export function createRoot(
  parent: Element | Document | ShadowRoot | DocumentFragment
): {
  render: (vnode: ComponentChild, replaceNode?: Element | Text ) => void;
  hydrate: (vnode: ComponentChild) => void;
};
```

> This is the bulk of the RFC. Explain the design in enough detail for somebody
> familiar with Preact to understand, and for somebody familiar with the
> implementation to implement. This should get into specifics and corner-cases,
> and include examples of how the feature is used. Any new terminology should be
> defined here.

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

> Optional, but suggested for first drafts. What parts of the design are still
> TBD?
