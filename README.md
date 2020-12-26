# Preact RFCs

This repository is a place to discuss major changes to Preact — where 'major'
means significant changes either to public interfaces or internal implementation
details, particularly those that could be controversial or involve breaking
changes.

Most changes don't need to go through the RFC (request for comments) process
outlined below and can rely on issues and pull requests.

A huge part of the value on an RFC is defining the problem clearly, collecting
use cases, showing how others have solved a problem, etc. Coming up with a
design is very iterative and only one part of the process. An RFC can provide
tremendous value without the design described in it being accepted.

[Pending RFC List](https://github.com/preactjs/rfcs/pulls)

Preact is still **developing** this process, and it is subject to change (or
even abandonment) as we gain experience with it.

## The RFC life-cycle

An RFC goes through the following stages:

- **Pending:** when the RFC is submitted as a PR.
- **Active:** when an RFC PR is merged and possibly undergoing implementation.
- **Landed:** when an RFCs proposed changes are shipped in an actual release.
- **Rejected:** when an RFC PR is closed without being merged.

## The process

In short, to get a major feature added to Preact, one usually first gets the RFC
merged into the RFC repo as a markdown file. At that point the RFC is 'active'
and may be implemented with the goal of eventual inclusion into Preact.

* Fork the RFC repo http://github.com/preactjs/rfcs

* Copy `0000-template.md` to `text/0000-my-feature.md` (where 'my-feature' is
  descriptive. Don't assign an RFC number yet).

* Fill in the RFC. Put care into the details: **RFCs that do not present
  convincing motivation, demonstrate understanding of the impact of the design,
  or are disingenuous about the drawbacks or alternatives tend to be
  poorly-received.**

* Submit a pull request. As a pull request the RFC will receive design feedback
  from the larger community, and the author should be prepared to revise it in
  response.

* Build consensus and integrate feedback. RFCs that have broad support are much
  more likely to make progress than those that don't receive any comments.

* Eventually, the team will decide whether the RFC is a candidate for inclusion
  in Preact.

* RFCs that are candidates for inclusion in Preact will enter a "final comment
  period" lasting 3 calendar days. The beginning of this period will be signaled
  with a comment and tag on the RFCs pull request. An RFC can be modified based
  upon feedback from the team and community. Significant modifications may
  trigger a new final comment period.

* An RFC may be rejected after public discussion has settled and comments have
  been made summarizing the rationale for rejection. A member of the team should
  then close the RFCs associated pull request.

* An RFC may be accepted at the close of its final comment period. A team member
  will merge the RFCs associated pull request, at which point the RFC will
  become 'active'.

## Active RFCs

Once an RFC becomes active, then authors may implement it and submit the feature
as a pull request to the Preact repo. Becoming 'active' is not a rubber stamp,
and in particular still does not mean the feature will ultimately be merged; it
does mean that the core team has agreed to it in principle and are amenable to
merging it.

Furthermore, the fact that a given RFC has been accepted and is 'active' implies
nothing about what priority is assigned to its implementation, nor whether
anybody is currently working on it.

Modifications to active RFCs can be done in followup PRs. We strive to write
each RFC in a manner that it will reflect the final design of the feature; but
the nature of the process means that we cannot expect every merged RFC to
actually reflect what the end result will be at the time of the next major
release; therefore we try to keep each RFC document somewhat in sync with the
language feature as planned, tracking such changes via followup pull requests to
the document.

## Implementing an RFC

The author of an RFC is not obligated to implement it. Of course, the RFC author
(like any other developer) is welcome to post an implementation for review after
the RFC has been accepted.

An active RFC should have the link to the implementation PR listed if there is
one. Feedback to the actual implementation should be conducted in the
implementation PR instead of the original RFC PR.

If you are interested in working on the implementation for an 'active' RFC, but
cannot determine if someone else is already working on it, feel free to ask
(e.g. by leaving a comment on the associated issue).

## Reviewing RFCs

Members of the team will attempt to review some set of open RFC pull requests on
a regular basis. If a core team member believes an RFC PR is ready to be
accepted into active status, they can approve the PR using GitHub's review
feature to signal their approval of the RFC.

We tend to do our thinking informally, in the open, when time allows. There are
a large number of community members relative to a small number of a core
contributors who have many responsibilities. You can help ensure your RFC is
reviewed in a timely manner by putting in the time to think through the various
details discussed in the template. It doesn't scale to push the thinking onto a
small number of core contributors. If reviewers raise an issue, don't dismiss it
as irrelevant, but take the time to provide examples or data explaining it and
coming up with ways that the design might be changed in response. Sometimes
answering a single question can be very time consuming (such as setting up a
benchmark), but discussions tend to stall out if concerns don't get thoroughly
addressed.

## Other RFC processes

Our RFC process is inspired by (which is to say shamelessly ripped off from)
similar processes in the community including:

- [React RFC process](https://github.com/reactjs/rfcs)
- [VueJS RFC process](https://github.com/vuejs/rfcs)
- [Svelte RFC process](https://github.com/sveltejs/rfcs)
- [Ember RFC process](https://github.com/emberjs/rfcs)
- [Rust RFC process](https://github.com/rust-lang/rfcs)
- [Yarn RFC process](https://github.com/yarnpkg/rfcs)
