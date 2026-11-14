# The Giant Swarm line of Helm

This repository is Team Bumblebee's fork of [helm/helm](https://github.com/helm/helm), whose Go SDK
(`helm.sh/helm/v4`) [giantswarm/agentlab](https://github.com/giantswarm/agentlab) installs the agent platform's charts
with. It carries upstream fixes agentlab needs before upstream releases them, and answers one question from one place:
**which Helm SDK does agentlab run, and why does it differ from upstream?**

## Branches

| Branch | What it is |
|---|---|
| `giantswarm` (default) | **The line**: the upstream release tag agentlab runs ("the pin") + the carried patches + this fork's own files (`FORK.md`, `fork/`, `renovate.json5`; upstream's `release.yml`, `scorecards.yml` and `stale.yaml` workflows removed, since the line publishes no release and no Scorecard result and closes no issue; upstream's `build-test.yml` runs on `giantswarm` too). Every change is a pull request against it. |
| `main` | upstream `main` at the time of the fork ("the mirror"); not consumed, not synced. |
| `upstream/<slug>` | a carried patch prepared for upstream: its one commit on the mirror's `main` |
| `fork/<topic>` | pull-request branches against `giantswarm` |

## Pin

Upstream **v4.2.4** (`3900f434`). The line moves to the next upstream release agentlab takes, with every carried patch
kept in effect on it or dropped once upstream contains it.

## Moving the pin

A pull request from `fork/v<version>` against `giantswarm` merges the upstream tag into the line
(`git merge --no-ff v<version>`), resolving conflicts so every carried patch stays in effect on the new upstream code
and the fork's own files stay as the line has them; `make test` passes before it opens. It lands with a merge commit,
so the new upstream tag is an ancestor of the line and the line's pseudo-versions count from it. The protected branch
is never force-pushed or rebased.

## Consumers

agentlab takes the line through a `replace` directive, keeping upstream's module path and release in its `require`,
pinned to a commit of `giantswarm` by its pseudo-version:

```
replace helm.sh/helm/v4 v4.2.4 => github.com/giantswarm/helm/v4 v4.2.5-0.<date>-<commit>
```

`go list -m github.com/giantswarm/helm/v4@<commit>` prints the pseudo-version. The module path in `go.mod` stays
`helm.sh/helm/v4`, so the line is only usable through `replace`. The `replace` names the upstream version it replaces,
so a bump of `helm.sh/helm/v4` leaves the line, and the consumer's tests show whether upstream carries the patches. The
`replace` goes away, and this line retires, at the first upstream release that contains every carried patch.

## Releases

None. A Go module needs no build or publish step, and a pseudo-version names exactly the commit of the protected
`giantswarm` branch a consumer runs, so the line cuts no tags of its own (an own tag would sit in upstream's `v4`
version space). Upstream tags on the line are upstream's, kept for the pseudo-versions.

## Carried patches

Each change the line carries has its own ledger file in [`fork/`](fork/); the directory listing is the set of carried
changes, and a file's name is the change's stable key. A change adds or edits only its own file, never this one. A
ledger file has a title naming the change and:

- **Status:** `carried`, or `dropped` once a re-pin onto an upstream release that contains it makes it redundant
- **Upstream:** the upstream pull request or issue and its state; until one is opened, "to file — prepared in the
  fork: branch `upstream/<slug>`", with the mirror commit the branch is based on and the upstream search
- **Our commit:** the commit on `giantswarm` and the pull request that landed it
- `## Change`: what the change does
- `## Reason`: why the line carries it

## Contributing

- **Upstream first, prepared here.** Before a change, search upstream's issues and pull requests, open and closed:
  work upstream already has in flight is supported with our evidence, not duplicated. Every behavioural change is
  prepared for upstream (see upstream's `CONTRIBUTING.md`) as one upstream-ready commit: upstream's conventions and
  tests, DCO-signed (`git commit -s`), no fork references, its message the upstream pull request's text. It is pushed
  as an `upstream/<slug>` branch on this repository, the commit on the mirror's `main`, and lands on `giantswarm` as a
  `git cherry-pick -x` of that commit with its ledger file. The ledger file's **Upstream** reads "to file — prepared in
  the fork: branch `upstream/<slug>`" until it is opened, and links the upstream pull request from then on.
- **A human opens it upstream.** The maintainers review the prepared branches in batches, decide what is useful to
  upstream and open it; the upstream pull request then names the fork commit. An agent never opens, comments on,
  reviews or closes a pull request or issue on the upstream repository.
- **Public only.** This repository is public: a ledger file, a commit message, a pull request and this file cite
  public issues and pull requests only; where the reasoning lives in a private tracker it is written out in a sentence
  instead. A pull request or commit message here names no upstream issue or pull request, so that nothing on this
  fork shows up on upstream's timeline: the ledger file carries those links.
- **Merging.** A carried patch is rebase-merged, so its upstream-ready commit lands as it is and falls away at the
  re-pin; a fork-only change (this file, `renovate.json5`, workflows) is squash-merged; a pin move lands with a merge
  commit.
- **Renovate is off** (`renovate.json5`): upstream owns the dependency graph, and dependencies move when the line is
  re-pinned.
