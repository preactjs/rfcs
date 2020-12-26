- Start Date: (fill me in with today's date, YYYY-MM-DD)
- Target Major Version: (e.g. 10.x / 11.x)
- RFC PR: (Fill this in once RFC PR is open)
- Implementation PR: (Fill this in if/when one exits)

# (RFC title goes here)

## Summary

Brief explanation of the feature.

## Basic example

If the proposal involves a new or changed API, include a basic code example.
Omit this section if it's not applicable.

## Motivation

Why are we doing this? What use cases does it support? What is the expected
outcome?

Please focus on explaining the motivation so that if this RFC is not accepted,
the motivation could be used to develop alternative solutions. In other words,
enumerate the constraints you are trying to solve without coupling them too
closely to the solution you have in mind.

Please provide specific examples. If you say "this would be more flexible" then
give an example of something that becomes easier. If you say "this would be make
it easier to do X" then give an example of what that looks like today and what's
hard about it.

Don't assume that others recognize the problem is one that needs to be solved.
Is there some concrete issue you cannot accomplish without this? What does it
look like to accomplish some set of goals today and how could that be improved?
Are there any workarounds that are necessary today? Are there open issues on
Github where people would be helped by this?

## Detailed design

This is the bulk of the RFC. Explain the design in enough detail for somebody
familiar with Preact to understand, and for somebody familiar with the
implementation to implement. This should get into specifics and corner-cases,
and include examples of how the feature is used. Any new terminology should be
defined here.

## Drawbacks

Why should we *not* do this? Please consider:

- implementation cost, both in term of code size, performance, and complexity
- whether the proposed feature can be implemented in user space
- the impact on teaching people Preact
- integration of this feature with other existing and planned features
- cost of migrating existing Preact applications (is it a breaking change?)

There are tradeoffs to choosing any path. Attempt to identify them here.

## Alternatives

What other designs have been considered? What is the impact of not doing this?

This section could also include prior art, that is, how other frameworks in the
same domain have solved this problem similarly or differently.

## Adoption strategy

If we implement this proposal, how will existing Preact developers adopt it? Is
this a breaking change? Can we write a codemod? Can we provide a runtime adapter
library for the original API it replaces? How will this affect other projects in
the Preact ecosystem? Should we coordinate with other projects or libraries?

## How we teach this

What names and terminology work best for these concepts and why? How is this
idea best presented? As a continuation of existing Preact patterns, or as a
wholly new one?

Would the acceptance of this proposal mean the Preact documentation must be
re-organized or altered? Does it change how Preact is taught to new developers
at any level?

How should this feature be taught to existing Preact developers?

## Unresolved questions

Optional, but suggested for first drafts. What parts of the design are still
TBD?
