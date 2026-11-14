# Judge a status wait by the statuses that ended it

- **Status:** carried
- **Upstream:** to file — prepared in the fork: branch `upstream/statuswait-late-event` (one commit on the mirror's
  `main` @ `c521d30b`). helm/helm searched 2026-11-14, open and closed (`statuswait`, `statusObserver`, the error text,
  kstatus waits, Kyverno ClusterPolicy): no issue or pull request covers an event that arrives after the wait has
  ended. Related: [helm/helm#32066](https://github.com/helm/helm/issues/32066), the early Current of a fresh custom
  resource, open without a fix; [helm/helm#32608](https://github.com/helm/helm/pull/32608),
  [#32586](https://github.com/helm/helm/pull/32586), [#32222](https://github.com/helm/helm/pull/32222),
  [#32616](https://github.com/helm/helm/pull/32616) and [#32909](https://github.com/helm/helm/pull/32909) are open and
  change the same file, the first two the same observer, for other causes.
- **Our commit:** `fix(kube): judge a status wait by the statuses that ended it`, a `git cherry-pick -x` of the
  upstream-ready commit, rebase-merged with this file in
  [giantswarm/helm#3](https://github.com/giantswarm/helm/pull/3)

## Change

`pkg/kube/statuswait.go`: the observer that ends a wait becomes a `statusObserver` type. When every resource has the
desired status it keeps a copy of all statuses, cancels the watch and ignores every later event; `wait` and
`waitForDelete` judge that copy, and the latest statuses only when the wait ended by timeout or cancellation. Two
tests in `statuswait_test.go` drive the waiter with a scripted `StatusWatcher` that delivers one more event after the
cancel; both fail without the change.

## Reason

agentlab's admission step installs a chart of Kyverno ClusterPolicies with the SDK's kstatus wait. kstatus reads a
policy Kyverno has not reconciled yet as Current, so the wait ends, and Kyverno's first status write, `Ready=False`
"Not Ready", is already in flight: the default watcher delivers it after the cancel and the waiter judged it, so a
fresh `agentlab up` failed now and then with `resource ClusterPolicy//<name> not ready. status: InProgress, message:
Not Ready`, seconds into its five-minute timeout. agentlab works around it by installing that release with
`--wait=hookOnly` and waiting for every policy's `Ready` condition itself
([giantswarm/agentlab#1287](https://github.com/giantswarm/agentlab/pull/1287), `HACKS.md` U50). With this change
the hooks-only wait can go; agentlab's own wait for `Ready` stays as long as kstatus reads a policy without a
condition as Current.
