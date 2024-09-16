<!--
**Note:** When your KEP is complete, all of these comment blocks should be removed.

To get started with this template:

- [ ] **Pick a hosting SIG.**
  Make sure that the problem space is something the SIG is interested in taking
  up. KEPs should not be checked in without a sponsoring SIG.
- [ ] **Create an issue in kubernetes/enhancements**
  When filing an enhancement tracking issue, please make sure to complete all
  fields in that template. One of the fields asks for a link to the KEP. You
  can leave that blank until this KEP is filed, and then go back to the
  enhancement and add the link.
- [ ] **Make a copy of this template directory.**
  Copy this template into the owning SIG's directory and name it
  `NNNN-short-descriptive-title`, where `NNNN` is the issue number (with no
  leading-zero padding) assigned to your enhancement above.
- [ ] **Fill out as much of the kep.yaml file as you can.**
  At minimum, you should fill in the "Title", "Authors", "Owning-sig",
  "Status", and date-related fields.
- [ ] **Fill out this file as best you can.**
  At minimum, you should fill in the "Summary" and "Motivation" sections.
  These should be easy if you've preflighted the idea of the KEP with the
  appropriate SIG(s).
- [ ] **Create a PR for this KEP.**
  Assign it to people in the SIG who are sponsoring this process.
- [ ] **Merge early and iterate.**
  Avoid getting hung up on specific details and instead aim to get the goals of
  the KEP clarified and merged quickly. The best way to do this is to just
  start with the high-level sections and fill out details incrementally in
  subsequent PRs.

Just because a KEP is merged does not mean it is complete or approved. Any KEP
marked as `provisional` is a working document and subject to change. You can
denote sections that are under active debate as follows:

```
<<[UNRESOLVED optional short context or usernames ]>>
Stuff that is being argued.
<<[/UNRESOLVED]>>
```

When editing KEPS, aim for tightly-scoped, single-topic PRs to keep discussions
focused. If you disagree with what is already in a document, open a new PR
with suggested changes.

One KEP corresponds to one "feature" or "enhancement" for its whole lifecycle.
You do not need a new KEP to move from beta to GA, for example. If
new details emerge that belong in the KEP, edit the KEP. Once a feature has become
"implemented", major changes should get new KEPs.

The canonical place for the latest set of instructions (and the likely source
of this file) is [here](/keps/NNNN-kep-template/README.md).

**Note:** Any PRs to move a KEP to `implementable`, or significant changes once
it is marked `implementable`, must be approved by each of the KEP approvers.
If none of those approvers are still appropriate, then changes to that list
should be approved by the remaining approvers and/or the owning SIG (or
SIG Architecture for cross-cutting KEPs).
-->
# KEP-4802: Graceful Node Shutdown for Windows Node

<!--
This is the title of your KEP. Keep it short, simple, and descriptive. A good
title can help communicate what the KEP is and should be considered as part of
any review.
-->

<!--
A table of contents is helpful for quickly jumping to sections of a KEP and for
highlighting any additional information provided beyond the standard KEP
template.

Ensure the TOC is wrapped with
  <code>&lt;!-- toc --&rt;&lt;!-- /toc --&rt;</code>
tags, and then generate with `hack/update-toc.sh`.
-->

<!-- toc -->
- [Release Signoff Checklist](#release-signoff-checklist)
- [Summary](#summary)
- [Motivation](#motivation)
  - [Goals](#goals)
  - [Non-Goals](#non-goals)
- [Proposal](#proposal)
  - [User Stories (Optional)](#user-stories-optional)
    - [Story 1](#story-1)
    - [Story 2](#story-2)
  - [Background on Windows Shutdown](#background-on-windows-shutdown)
  - [Implementation](#implementation)
  - [Notes/Constraints/Caveats (Optional)](#notesconstraintscaveats-optional)
  - [Risks and Mitigations](#risks-and-mitigations)
- [Design Details](#design-details)
  - [Test Plan](#test-plan)
      - [Prerequisite testing updates](#prerequisite-testing-updates)
      - [Unit tests](#unit-tests)
      - [Integration tests](#integration-tests)
      - [e2e tests](#e2e-tests)
  - [Graduation Criteria](#graduation-criteria)
    - [Alpha Graduation](#alpha-graduation)
    - [Alpha -&gt; Beta Graduation](#alpha---beta-graduation)
    - [Beta -&gt; GA Graduation](#beta---ga-graduation)
  - [Upgrade / Downgrade Strategy](#upgrade--downgrade-strategy)
  - [Version Skew Strategy](#version-skew-strategy)
- [Production Readiness Review Questionnaire](#production-readiness-review-questionnaire)
  - [Feature Enablement and Rollback](#feature-enablement-and-rollback)
  - [Rollout, Upgrade and Rollback Planning](#rollout-upgrade-and-rollback-planning)
  - [Monitoring Requirements](#monitoring-requirements)
  - [Dependencies](#dependencies)
  - [Scalability](#scalability)
  - [Troubleshooting](#troubleshooting)
- [Implementation History](#implementation-history)
- [Drawbacks](#drawbacks)
- [Alternatives](#alternatives)
- [Infrastructure Needed (Optional)](#infrastructure-needed-optional)
<!-- /toc -->

## Release Signoff Checklist

<!--
**ACTION REQUIRED:** In order to merge code into a release, there must be an
issue in [kubernetes/enhancements] referencing this KEP and targeting a release
milestone **before the [Enhancement Freeze](https://git.k8s.io/sig-release/releases)
of the targeted release**.

For enhancements that make changes to code or processes/procedures in core
Kubernetes—i.e., [kubernetes/kubernetes], we require the following Release
Signoff checklist to be completed.

Check these off as they are completed for the Release Team to track. These
checklist items _must_ be updated for the enhancement to be released.
-->

Items marked with (R) are required *prior to targeting to a milestone / release*.

- [x] (R) Enhancement issue in release milestone, which links to KEP dir in [kubernetes/enhancements] (not the initial KEP PR)
- [x] (R) KEP approvers have approved the KEP status as `implementable`
- [x] (R) Design details are appropriately documented
- [x] (R) Test plan is in place, giving consideration to SIG Architecture and SIG Testing input
- [x] (R) Graduation criteria is in place
- [x] (R) Production readiness review completed
- [ ] Production readiness review approved
- [ ] "Implementation History" section is up-to-date for milestone
- [ ] User-facing documentation has been created in [kubernetes/website], for publication to [kubernetes.io]
- [ ] Supporting documentation—e.g., additional design documents, links to mailing list discussions/SIG meetings, relevant PRs/issues, release notes

<!--
**Note:** This checklist is iterative and should be reviewed and updated every time this enhancement is being considered for a milestone.
-->

[kubernetes.io]: https://kubernetes.io/
[kubernetes/enhancements]: https://git.k8s.io/enhancements
[kubernetes/kubernetes]: https://git.k8s.io/kubernetes
[kubernetes/website]: https://git.k8s.io/website

## Summary

<!--
This section is incredibly important for producing high-quality, user-focused
documentation such as release notes or a development roadmap. It should be
possible to collect this information before implementation begins, in order to
avoid requiring implementors to split their attention between writing release
notes and implementing the feature itself. KEP editors and SIG Docs
should help to ensure that the tone and content of the `Summary` section is
useful for a wide audience.

A good summary is probably at least a paragraph in length.

Both in this section and below, follow the guidelines of the [documentation
style guide]. In particular, wrap lines to a reasonable length, to make it
easier for reviewers to cite specific portions, and to minimize diff churn on
updates.

[documentation style guide]: https://github.com/kubernetes/community/blob/master/contributors/guide/style-guide.md
-->
As specified in [kep2000](https://github.com/zylxjtu/enhancements/tree/master/keps/sig-node/2000-graceful-node-shutdown),
Kubelet should be aware of node shutdown and trigger graceful shutdown of pods
during a machine shutdown.

Node graceful shutdown has been available in Linux, this is to implement a comparable feature for windows node
## Motivation

<!--
This section is for explicitly listing the motivation, goals and non-goals of
this KEP.  Describe why the change is important and the benefits to users. The
motivation section can optionally provide links to [experience reports] to
demonstrate the interest in a KEP within the wider Kubernetes community.

[experience reports]: https://github.com/golang/go/wiki/ExperienceReports
-->

Users and cluster administrators expect that pods will adhere to expected pod
lifecycle including pod termination. Currently, when a node shuts down, pods do
not follow the expected pod termination lifecycle and are not terminated
gracefully which can cause issues for some workloads. This KEP aims to address
this problem by making the kubelet aware of the underlying node shutdown.
Kubelet will propagate this signal to pods ensuring they can shutdown as
gracefully as possible.

### Goals

<!--
List the specific goals of the KEP. What is it trying to achieve? How will we
know that this has succeeded?
-->

*   Make kubelet aware of underlying node shutdown event and trigger pod
    termination with sufficient grace period to shutdown properly
*   Handle node shutdown in cloud-provider agnostic way
*   Introduce minimal shutdown delay in order to shutdown node soon as possible
    (but not sooner)
*   Focus on handling shutdown on machines which kubelet itself runs as a windows service

### Non-Goals

<!--
What is out of scope for this KEP? Listing non-goals helps to focus discussion
and make progress.
-->

*   Let users modify or change existing pod lifecycle or introduce new inner
    pod depencides / shutdown ordering
*   Support kubelet running as a non-windows service
*   Support Cancellation of windows shutdown
*   Provide guarantee to handle all cases of graceful node shutdown, for
    example abrupt shutdown or sudden power cable pull can’t result in graceful
    shutdown
 *   Support the cancellation of shutdown  

## Proposal

<!--
This is where we get down to the specifics of what the proposal actually is.
This should have enough detail that reviewers can understand exactly what
you're proposing, but should not include things like API designs or
implementation. The "Design Details" section below is for the real
nitty-gritty.
-->

### User Stories (Optional)

<!--
Detail the things that people will be able to do if this KEP is implemented.
Include as much detail as possible so that people can understand the "how" of
the system. The goal here is to make this feel real for users without getting
bogged down.
-->

#### Story 1

*   As a cluster administrator, I can configure the nodes in my cluster to
    allocate X seconds for my pods to terminate gracefully during a node
    shutdown

#### Story 2

*   As a developer I can expect that my pods will terminate gracefully during
    node shutdowns

### Background on Windows Shutdown

In the context of this KEP, shutdown is referred to as shutdown of the
underlying windows machine. A shutdown can be initiated via a
variety of methods for example:

1. `shutdown /t 0`
2. `shutdown /t 30` `#schedule a delayed shutdown in 30 seconds`
3. Physically pressing the power button on the machine
4. If a machine is a VM, the underlying hypervisor can press the “virtual”
   power button
5. For a cloud instance, stopping the instance via Cloud API, e.g. via `az vm stop -g xx -n xx`.
   Depending on the cloud provider, this may result in virtual power button press by the underlying hypervisor.

In the context of kubernetes on windows node, system is not
aware of the pods and containers running on the machine and will simply
kill them as regular windows processes, that's the reason that we will need
to monitoring the system shutdown and behave accrodingly.

The high-level process of detecting a system shutdown on windows is the same for all types of Windows applications and services. To detect a shutdown, 
we create a callback function and register it in the system. When a certain Windows event occurs, the system calls the callback function, transferring 
information about the event via input parameters. The callback function then analyzes the input parameter data, and executes code accordingly.

There’s no one way to make all types of applications (console, GUI and windows services) detect a node shutdown, different types of applications differ 
in their syntax and functionality.

For general windows console application, it can process the "CTRL_SHUTDOWN_EVENT", but according to [HandlerRoutine callback function](https://learn.microsoft.com/en-us/windows/console/handlerroutine#parameters)
and the [SetConsoleCtrlHandler function](https://learn.microsoft.com/en-us/windows/console/setconsolectrlhandler?redirectedfrom=MSDN) 
`Console functions, or any C run-time functions that call console functions, may not work reliably during processing of any of the three signals mentioned previously. 
The reason is that some or all of the internal console cleanup routines may have been called before executing the process signal handler`.

While for windows service programs, we will have more freedom to register a service control handler to monitoring various shutdown related event 
such as "SERVICE_CONTROL_PRESHUTDOWN", "SERVICE_CONTROL_SHUTDOWN". So in order to configure and process the (pre)shutdown event from windows system, 
register the kubelet application as a windows service program will be the most straightforward way.

Aso from the same document [SetConsoleCtrlHandler function](https://learn.microsoft.com/en-us/windows/console/setconsolectrlhandler?redirectedfrom=MSDN),
It appears that "SERVICE_CONTROL_PRESHUTDOWN" is more flexible than "SERVICE_CONTROL_SHUTDOWN" for a windows service to monitor and control

### Implementation
Similar with the linux case, "nodeshutdown_manager_windows.go" will be added to replace the current
dumb implementation for windows, it will act as the main role in the node shutdown scenario and go through similar 
initialization/start process during kublet start up.

Also, we will update the windows service package to "AcceptPreShutdown" and add a pre-shutdown handler to process
the "SERVICE_CONTROL_PRESHUTDOWN" event. This pre-shutdown handler will have an interface "ProcessShutdownEvent",
which will be implemented by the "nodeshutdown_manager_windows"

The entry point of running the kublet as a windows service is "InitService", in which, we can initialize and register 
the servcie control handler, the above "pre-shutdown handler" is part of this service control handler.

At this "InitService" stage, the kubelet has not read the kubelet configs/flags yet, so the "nodeshutdown_manager_windows", 
has not been instantiated, we will then set the "preshutdownhander" to be NIL, so will ignore the preshutdown event if any. 
This should be safe enough as the kubelset itself has not been started, let alone the pods it managed, so there is nothing 
need to do for shutting down at this stage.

When kubelet proceeds with starting up and initialize the "nodeshutdown_manager_window", it will set the servic control ehandler’s 
pre-shutdownhandler to the "nodeshutdown_manager_window" and is now ready to monitor and process the preshutdown event

We will also add function of "UpdatePreShutdownInfo" and "QueryPreShutdownInfo" to windows service package, these functions will be 
used by the "nodeshutdown_manager" to query and update the preshtudown timeout.

If we will take the 'pre-shutdown timeout value as pariale with the 'InhibitDelayMaxSec` in linux,
then the other parts of implementation will be pretty much similar with what linux has [implemented](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2000-graceful-node-shutdown#implementation)

The poc of the implementation can be found [here](https://github.com/zylxjtu/kubernetes/commit/854ea4bde88c0905241b43f5f80d470967bb909f#diff-8494875e9a1e884afd36f32fa90dceb2b827616f1e981ecb83a12901262214c7)

### Notes/Constraints/Caveats (Optional)

<!--
What are the caveats to the proposal?
What are some important details that didn't come across above?
Go in to as much detail as necessary here.
This might be a good place to talk about core concepts and how they relate.
-->

### Risks and Mitigations

<!--
What are the risks of this proposal, and how do we mitigate? Think broadly.
For example, consider both security and how this will impact the larger
Kubernetes ecosystem.

How will security be reviewed, and by whom?

How will UX be reviewed, and by whom?

Consider including folks who also work outside the SIG or subproject.
-->

Please refer to the the [part](https://github.com/kubernetes/enhancements/tree/master/keps/sig-node/2000-graceful-node-shutdown#notesconstraintscaveats-optional) in KEP-2000

## Design Details

<!--
This section should contain enough information that the specifics of your
change are understandable. This may include API specs (though not always
required) or even code snippets. If there's any ambiguity about HOW your
proposal will be implemented, this is the place to discuss them.
-->

Use the same Kubelet Config as in linux.
```
type KubeletConfiguration struct {
    ...
    ShutdownGracePeriod metav1.Duration
    ShutdownGracePeriodCriticalPods metav1.Duration
}
```

Communication with service control manager will make use of golang.org/x/sys/windows/svc packag,
which is already included in [vendor](https://github.com/kubernetes/kubernetes/tree/release-1.19/vendor/golang.org/x/sys/windows)


Termination of pods will make use of the existing
[killPod](https://github.com/kubernetes/kubernetes/blob/release-1.19/pkg/kubelet/pod_workers.go#L292) function
from the `kubelet` package and specify the appropriate `gracePeriodOverride` as
necessary.

### Test Plan

<!--
**Note:** *Not required until targeted at a release.*
The goal is to ensure that we don't accept enhancements with inadequate testing.

All code is expected to have adequate tests (eventually with coverage
expectations). Please adhere to the [Kubernetes testing guidelines][testing-guidelines]
when drafting this test plan.

[testing-guidelines]: https://git.k8s.io/community/contributors/devel/sig-testing/testing.md
-->

[x] I/we understand the owners of the involved components may require updates to
existing tests to make this code solid enough prior to committing the changes necessary
to implement this enhancement.

##### Prerequisite testing updates

<!--
Based on reviewers feedback describe what additional tests need to be added prior
implementing this enhancement to ensure the enhancements have also solid foundations.
-->

##### Unit tests

<!--
In principle every added code should have complete unit test coverage, so providing
the exact set of tests will not bring additional value.
However, if complete unit test coverage is not possible, explain the reason of it
together with explanation why this is acceptable.
-->
It does not seem possible to unit test the windows logic of setting service handler and update registry keys

The main logic of kill pod is the same as what linux did and there has been unit tests covered there.
<!--
Additionally, for Alpha try to enumerate the core package you will be touching
to implement this enhancement and provide the current unit coverage for those
in the form of:
- <package>: <date> - <current test coverage>
The data can be easily read from:
https://testgrid.k8s.io/sig-testing-canaries#ci-kubernetes-coverage-unit

This can inform certain test coverage improvements that we want to do before
extending the production code to implement this enhancement.
-->

- `<package>`: `<date>` - `<test coverage>`

##### Integration tests

<!--
Integration tests are contained in k8s.io/kubernetes/test/integration.
Integration tests allow control of the configuration parameters used to start the binaries under test.
This is different from e2e tests which do not allow configuration of parameters.
Doing this allows testing non-default options and multiple different and potentially conflicting command line options.
-->

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html
-->

- <test>: <link to test coverage>

##### e2e tests

<!--
This question should be filled when targeting a release.
For Alpha, describe what tests will be added to ensure proper quality of the enhancement.

For Beta and GA, add links to added tests together with links to k8s-triage for those tests:
https://storage.googleapis.com/k8s-triage/index.html

We expect no non-infra related flakes in the last month as a GA graduation criteria.
-->
*   New E2E tests to validate node graceful shutdown.
    *   Shutdown grace period unspecified, feature is not active
    *   Pod’s ExecStop and SIGTERM handlers are given gracePeriodSeconds for
        case when gracePeriodSeconds <= kubeletConfig.ShutdownGracePeriod
    *   Pod’s ExecStop and SIGTERM handlers are given
        kubeletConfig.ShutdownGracePeriod for case when gracePeriodSeconds >
        kubeletConfig.ShutdownGracePeriod

### Graduation Criteria

<!--
**Note:** *Not required until targeted at a release.*

Define graduation milestones.

These may be defined in terms of API maturity, or as something else. The KEP
should keep this high-level with a focus on what signals will be looked at to
determine graduation.

Consider the following in developing the graduation criteria for this enhancement:
- [Maturity levels (`alpha`, `beta`, `stable`)][maturity-levels]
- [Deprecation policy][deprecation-policy]

Clearly define what graduation means by either linking to the [API doc
definition](https://kubernetes.io/docs/concepts/overview/kubernetes-api/#api-versioning)
or by redefining what graduation means.

In general we try to use the same stages (alpha, beta, GA), regardless of how the
functionality is accessed.

[maturity-levels]: https://git.k8s.io/community/contributors/devel/sig-architecture/api_changes.md#alpha-beta-and-stable-versions
[deprecation-policy]: https://kubernetes.io/docs/reference/using-api/deprecation-policy/

Below are some examples to consider, in addition to the aforementioned [maturity levels][maturity-levels].

#### Alpha -> Beta Graduation

- Gather feedback from developers and surveys
- Complete features A, B, C
- Tests are in Testgrid and linked in KEP

#### Beta -> GA Graduation

- N examples of real-world usage
- N installs
- More rigorous forms of testing—e.g., downgrade tests and scalability tests
- Allowing time for feedback

**Note:** Generally we also wait at least two releases between beta and
GA/stable, because there's no opportunity for user feedback, or even bug reports,
in back-to-back releases.

#### Removing a Deprecated Flag

- Announce deprecation and support policy of the existing flag
- Two versions passed since introducing the functionality that deprecates the flag (to address version skew)
- Address feedback on usage/changed behavior, provided on GitHub issues
- Deprecate the flag

**For non-optional features moving to GA, the graduation criteria must include
[conformance tests].**

[conformance tests]: https://git.k8s.io/community/contributors/devel/sig-architecture/conformance-tests.md
-->

#### Alpha Graduation

* Implemented the feature for windows service only

* Investigate how e2e tests can be implemented (e.g. may need to create fake
  shutdown event)

#### Alpha -> Beta Graduation

* Addresses feedback from alpha testers
* Sufficient E2E and unit testing

#### Beta -> GA Graduation

* Addresses feedback from beta
* Sufficient number of users using the feature
* Confident that no further API / kubelet config configuration options changes are needed
* Close on any remaining open issues & bugs

### Upgrade / Downgrade Strategy

<!--
If applicable, how will the component be upgraded and downgraded? Make sure
this is in the test plan.

Consider the following in developing an upgrade/downgrade strategy for this
enhancement:
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to maintain previous behavior?
- What changes (in invocations, configurations, API use, etc.) is an existing
  cluster required to make on upgrade, in order to make use of the enhancement?
-->

n/a

### Version Skew Strategy

<!--
If applicable, how will the component handle version skew with other
components? What are the guarantees? Make sure this is in the test plan.

Consider the following in developing a version skew strategy for this
enhancement:
- Does this enhancement involve coordinating behavior in the control plane and
  in the kubelet? How does an n-2 kubelet without this feature available behave
  when this feature is used?
- Will any other components on the node change? For example, changes to CSI,
  CRI or CNI may require updating that component before the kubelet.
-->

n/a

## Production Readiness Review Questionnaire

<!--

Production readiness reviews are intended to ensure that features merging into
Kubernetes are observable, scalable and supportable; can be safely operated in
production environments, and can be disabled or rolled back in the event they
cause increased failures in production. See more in the PRR KEP at
https://git.k8s.io/enhancements/keps/sig-architecture/20190731-production-readiness-review-process.md.

The production readiness review questionnaire must be completed for features in
v1.19 or later, but is non-blocking at this time. That is, approval is not
required in order to be in the release.

In some cases, the questions below should also have answers in `kep.yaml`. This
is to enable automation to verify the presence of the review, and to reduce review
burden and latency.

The KEP must have a approver from the
[`prod-readiness-approvers`](http://git.k8s.io/enhancements/OWNERS_ALIASES)
team. Please reach out on the
[#prod-readiness](https://kubernetes.slack.com/archives/CPNHUMN74) channel if
you need any help or guidance.

-->

### Feature Enablement and Rollback

_This section must be completed when targeting alpha to a release._

* **How can this feature be enabled / disabled in a live cluster?**
  - [X] Feature gate (also fill in values in `kep.yaml`)
    - Feature gate name: `GracefulNodeShutdown`
    - Components depending on the feature gate:
      - `kubelet`
  - [ ] Other
    - Describe the mechanism:
    - Will enabling / disabling the feature require downtime of the control
      plane?
      - no
    - Will enabling / disabling the feature require downtime or reprovisioning
      of a node? (Do not assume `Dynamic Kubelet Config` feature is enabled).
      - yes (will require restart of kubelet)

* **Does enabling the feature change any default behavior?**
  Any change of default behavior may be surprising to users or break existing
  automations, so be extremely careful here.

    * The main behavior change is that during a node shutdown, pods running on
      the node will be terminated gracefully.

* **Can the feature be disabled once it has been enabled (i.e. can we roll back
  the enablement)?**
  Also set `disable-supported` to `true` or `false` in `kep.yaml`.
  Describe the consequences on existing workloads (e.g., if this is a runtime
  feature, can it break the existing applications?).

    * Yes, the feature can be disabled by either disabling the feature gate, or
      setting `kubeletConfig.ShutdownGracePeriod` to 0 seconds.

* **What happens if we reenable the feature if it was previously rolled back?**

    * Kubelet will attempt to perform graceful termination of pods during a
        node shutdown.

* **Are there any tests for feature enablement/disablement?**
  The e2e framework does not currently support enabling or disabling feature
  gates. However, unit tests in each component dealing with managing data, created
  with and without the feature, are necessary. At the very least, think about
  conversion tests if API types are being modified.

    *   n/a

### Rollout, Upgrade and Rollback Planning

_This section must be completed when targeting beta graduation to a release._

* **How can a rollout fail? Can it impact already running workloads?**
  Try to be as paranoid as possible - e.g., what if some components will restart
   mid-rollout?

This feature should not impact rollouts.

* **What specific metrics should inform a rollback?**

N/A.

* **Were upgrade and rollback tested? Was the upgrade->downgrade->upgrade path tested?**
  Describe manual testing that was done and the outcomes.
  Longer term, we may want to require automated upgrade/rollback tests, but we
  are missing a bunch of machinery and tooling and can't do that now.

The feature is part of kubelet config so updating kubelet config should
enable/disable the feature; upgrade/downgrade is N/A.

* **Is the rollout accompanied by any deprecations and/or removals of features, APIs,
fields of API types, flags, etc.?**
  Even if applying deprecation policies, they may still surprise some users.

No.

### Monitoring Requirements

_This section must be completed when targeting beta graduation to a release._

* **How can an operator determine if the feature is in use by workloads?**
  Ideally, this should be a metric. Operations against the Kubernetes API (e.g.,
  checking if there are objects with field X set) may be a last resort. Avoid
  logs or events for this purpose.

Check if the feature gate and kubelet config settings are enabled on a node.

* **What are the SLIs (Service Level Indicators) an operator can use to determine
the health of the service?**
  - [ ] Metrics
    - Metric name:
    - [Optional] Aggregation method:
    - Components exposing the metric:
  - [ ] Other (treat as last resort)
    - Details:

N/A

* **What are the reasonable SLOs (Service Level Objectives) for the above SLIs?**
  At a high level, this usually will be in the form of "high percentile of SLI
  per day <= X". It's impossible to provide comprehensive guidance, but at the very
  high level (needs more precise definitions) those may be things like:
  - per-day percentage of API calls finishing with 5XX errors <= 1%
  - 99% percentile over day of absolute value from (job creation time minus expected
    job creation time) for cron job <= 10%
  - 99,9% of /health requests per day finish with 200 code

N/A.

* **Are there any missing metrics that would be useful to have to improve observability
of this feature?**
  Describe the metrics themselves and the reasons why they weren't added (e.g., cost,
  implementation difficulties, etc.).

N/A.

### Dependencies

_This section must be completed when targeting beta graduation to a release._

* **Does this feature depend on any specific services running in the cluster?**
  Think about both cluster-level services (e.g. metrics-server) as well
  as node-level agents (e.g. specific version of CRI). Focus on external or
  optional services that are needed. For example, if this feature depends on
  a cloud provider API, or upon an external software-defined storage or network
  control plane.

  For each of these, fill in the following—thinking about running existing user workloads
  and creating new ones, as well as about cluster-level services (e.g. DNS):
  - [Dependency name]
    - Usage description:
      - Impact of its outage on the feature:
      - Impact of its degraded performance or high-error rates on the feature:

No, this feature doesn't depend on any specific services running the cluster.
It only depends on systemd running on the node itself.

### Scalability

_For alpha, this section is encouraged: reviewers should consider these questions
and attempt to answer them._

_For beta, this section is required: reviewers must answer these questions._

_For GA, this section is required: approvers should be able to confirm the
previous answers based on experience in the field._

* **Will enabling / using this feature result in any new API calls?**
  Describe them, providing:
  - API call type (e.g. PATCH pods)
  - estimated throughput
  - originating component(s) (e.g. Kubelet, Feature-X-controller)
  focusing mostly on:
  - components listing and/or watching resources they didn't before
  - API calls that may be triggered by changes of some Kubernetes resources
    (e.g. update of object X triggers new updates of object Y)
  - periodic API calls to reconcile state (e.g. periodic fetching state,
    heartbeats, leader election, etc.)

No.

* **Will enabling / using this feature result in introducing new API types?**
  Describe them, providing:
  - API type
  - Supported number of objects per cluster
  - Supported number of objects per namespace (for namespace-scoped objects)

No.

* **Will enabling / using this feature result in any new calls to the cloud
provider?**

No.

* **Will enabling / using this feature result in increasing size or count of
the existing API objects?**
  Describe them, providing:
  - API type(s):
  - Estimated increase in size: (e.g., new annotation of size 32B)
  - Estimated amount of new objects: (e.g., new Object X for every existing Pod)

No.

* **Will enabling / using this feature result in increasing time taken by any
operations covered by [existing SLIs/SLOs]?**
  Think about adding additional work or introducing new steps in between
  (e.g. need to do X to start a container), etc. Please describe the details.

No.

* **Will enabling / using this feature result in non-negligible increase of
resource usage (CPU, RAM, disk, IO, ...) in any components?**
  Things to keep in mind include: additional in-memory state, additional
  non-trivial computations, excessive access to disks (including increased log
  volume), significant amount of data sent and/or received over network, etc.
  This through this both in small and large cases, again with respect to the
  [supported limits].

No.

### Troubleshooting

The Troubleshooting section currently serves the `Playbook` role. We may consider
splitting it into a dedicated `Playbook` document (potentially with some monitoring
details). For now, we leave it here.

_This section must be completed when targeting beta graduation to a release._

* **How does this feature react if the API server and/or etcd is unavailable?**

The feature does not depend on the API server / etcd.

* **What are other known failure modes?**
  For each of them, fill in the following information by copying the below template:
  - [Failure mode brief description]
    - Detection: How can it be detected via metrics? Stated another way:
      how can an operator troubleshoot without logging into a master or worker node?
    - Mitigations: What can be done to stop the bleeding, especially for already
      running user workloads?
    - Diagnostics: What are the useful log messages and their required logging
      levels that could help debug the issue?
      Not required until feature graduated to beta.
    - Testing: Are there any tests for failure mode? If not, describe why.

* **What steps should be taken if SLOs are not being met to determine the problem?**

[supported limits]: https://git.k8s.io/community//sig-scalability/configs-and-limits/thresholds.md
[existing SLIs/SLOs]: https://git.k8s.io/community/sig-scalability/slos/slos.md#kubernetes-slisslos

N/A.

## Implementation History

<!--
Major milestones in the lifecycle of a KEP should be tracked in this section.
Major milestones might include:
- the `Summary` and `Motivation` sections being merged, signaling SIG acceptance
- the `Proposal` section being merged, signaling agreement on a proposed design
- the date implementation started
- the first Kubernetes release where an initial version of the KEP was available
- the version of Kubernetes where the KEP graduated to general availability
- when the KEP was retired or superseded
-->

*   2024-08-31 - [Initial KEP approved](https://github.com/kubernetes/enhancements/pull/2001)
*   
## Drawbacks

<!--
Why should this KEP _not_ be implemented?
-->

## Alternatives

<!--
What other approaches did you consider, and why did you rule them out? These do
not need to be as detailed as the proposal, but should include enough
information to express the idea and why it was not acceptable.
-->

* No need to run kubelet itself as a windows service but as a general windows console application
    * General windows application does not have much control on seting of the shutdown timeout value, and
      does not seem reliable to receive the shutdown event. A common practice of running kubelet on windows
      node right now is through an external service control manager such as nssm. But no matter what the
      external service control manager will be, if the kubelet executable will not abel to get access to its
      service control handler, it will not be able to apply the corresponding logic to monitor and response to
      the corresponding shutdown event.
* Use RegKey WaitToKillServiceTimeout to control the shutdown time out value
    * As discussed in the Background part, windows does not prefer to use this RegKey to update the shutdown
      timeout value, which will have a global affect on the services running on the host.

## Infrastructure Needed (Optional)

<!-- Use this section if you need things from the project/SIG. Examples include
a new subproject, repos requested, or GitHub details. Listing these here allows
a SIG to get the process for these resources started right away.  -->

 
 


