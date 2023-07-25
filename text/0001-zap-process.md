- ZAP PR:
  [zulip/zulip-architecture#1](https://github.com/zulip/zulip-architecture/pull/1)

# Summary

The Zulip architecture proposal (ZAP) process gives the Zulip core
team a moderately more formal way to propose, collaborate on, and
build consensus for architectural design proposals.

# Motivation

There are several reasons a given proposed change to the Zulip architecture
can benefit from an explicit design proposal through the ZAP process.

- Some categories of proposed changes to the Zulip architecture pose
  backwards compatibility concerns if we get them wrong.

  - We aim to support APIs for as long as reasonably possible, to avoid
    breaking clients using in the wild. We use the [api design
    channel](https://chat.zulip.org/#topics/channel/378-api-design) for
    tactical API design decision-making, but some big decisions that may
    be hard to change later deserve a more deliberative process.
  - We should ensure that data persisted to the database is structured in a way
    that will remain useful forever, to avoid expensive future migrations (which
    themselves need to be maintained forever).

  These changes will benefit from a design proposal phase separate from the
  implementation phase, where we work out a good design with an eye toward future
  maintainability before the details are set in stone.

- Some proposed changes may be controversial or involve deciding
  between a number of alternatives with different tradeoffs. These
  changes will benefit from discussion between all stakeholders, and a
  lightweight mechanism to record and collect the consensus decisions
  reached in those discussions, so that implementation can proceed in
  a consistent direction.

- Some proposed changes have a large enough scope that the
  implementation work is expected to be split over many pull requests
  over a long period of time. These changes will benefit from a
  detailed design document that guides the overall implementation plan
  and decreases the likelihood that earlier parts of the work need to
  be redone.

# Detailed design

When proposing a change that falls into the above categories or might otherwise
benefit from a formalize design process, after preliminary discussion in our
[development community](https://zulip.com/development-community/), a developer
may choose to open a Zulip architecture proposal (ZAP).

A ZAP is a pull request in this `zulip-architecture` repository that adds a
Markdown document named like `text/0000-my-feature.md`. The numerical ID should
be initialized with the placeholder `0000`, and replaced with the pull request
number after the pull request is opened.

The pull request description should link to the rendered Markdown document from
the author's branch. The Markdown document should begin with a link back to the
GitHub pull request page, and proceed with a suggested organization into
sections such as “Summary”, “Motivation”, “Detailed design”, “Alternatives”,
“Security implications”, “Future directions”, and “Previous discussion”, as
applicable.

Each new ZAP pull request should be posted to an appropriate stream in the
development community for discussion. Discussion can happen in the development
community or in comments/reviews on the pull request. Any discussion that
happens outside the pull request should ideally be linked and summarized on the
pull request to keep a record of it. The pull request will remain open while
discussion is ongoing, and may be updated as desired.

If the Zulip team reaches a consensus (via the usual decision-making conventions
of the Zulip project, which are deliberately left unspecified here) that the
proposal should be accepted, the pull request will be merged. If consensus is
that the proposal should be rejected, the pull request will be closed, and if
consensus is that it needs modifications, the proposal may remain open while
being edited, or be closed in favor of a different pull request.

The documents in `zulip-architecture` will be maintained to reflect
the current consensus of the project. Typically this entails:

- We expect to routinely edit ZAPs to link to issues/PRs related to
  the ZAP. When the implementation is complete, we'll edit the
  metadata at the top of the ZAP to say so, and to link to the
  relevant subsystem doc and/or API docs.
- When we change our mind about something while an accepted ZAP is
  still in the process of being implemented, we'll typically edit the
  ZAP to reflect the design change. (This helps avoid wasting time
  during the implementation phase).
- Once implementation is complete, we generally won't try to update
  ZAPs to match the current state of the architecture. The ZAP remains
  a record of what we were thinking then; people finding the ZAP and
  interested in the up-to-date API are advised to follow the links to
  current docs.
- If we subsequently make a major overhaul — roughly, something that
  calls for a ZAP of its own — then we go back and add a link at the
  top of the older ZAP to point to the new one, a bit like the
  references at the top of old RFCs.

# Alternatives

Most design decisions in Zulip have been made in the
[development community](https://zulip.com/development-community/) or within the
GitHub pull requests for the implementation of individual Zulip features. We
expect that most smaller decisions will continue to be made in those ways, and
the ZAP process will be used in planning more substantial changes that will
benefit from it.

Some design proposals have been drafted on other platforms such as Dropbox
Paper, but these are not accessible to our open source contributors outside the
core team and aren't collected in a useful way.

We considered writing design proposals within the Zulip
[developer documentation](https://zulip.readthedocs.io/) which lives in the same
repository as the server code. However, the developer documentation is intended
to document Zulip as currently implemented; it has different maintenance and
lifecycle needs from proposals that may pertain to a future or past
implementation changes.

Where appropriate, content can be copied from a ZAP into the developer
documentation once the ZAP is implemented; but rarely will it make
sense for a ZAP document to be copied in its entirely, since the
audience and goals are different.

The ZAP process is inspired by the
[Rust RFC process](https://github.com/rust-lang/rfcs), which itself takes
inspiration from the [Python PEP process](https://peps.python.org/).

# Previous discussion

- [#general > Why use both Memcache and Redis?](https://chat.zulip.org/#narrow/stream/2-general/topic/Why.20use.20both.20Memcache.20and.20Redis.3F/near/1002790),
  2020-08-28
- [#api design > end-to-end encryption protocols](https://chat.zulip.org/#narrow/stream/378-api-design/topic/end-to-end.20encryption.20protocols/near/1611909),
  2023-07-18
- [#api design > ZAP](https://chat.zulip.org/#narrow/stream/378-api-design/topic/ZAP/near/1613274),
  2023-07-20
