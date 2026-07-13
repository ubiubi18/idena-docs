---
hide_title: true
title: IIP-14
sidebar_label: IIP-14 Repeated reported-flip penalty
---

## IIP-14: Repeated reported-flip penalty

`Author`: ubiubi18

`Description`: Apply an identity-state penalty when two or more of an identity's flips are reported in one validation ceremony.

`Status`: Draft

`Type`: Standard

`Created`: 2025-08-05

`Discussion`: https://github.com/idena-network/idena-docs/discussions/187

`Translations`:

## Abstract

This proposal adds an identity-state penalty for publishing at least two flips that receive the protocol's final `GradeReported` grade during the same validation ceremony. An identity whose pre-validation state is `Newbie` is killed. An identity whose pre-validation state is `Verified` or `Human` is suspended, unless the existing validation rules already kill it. A single reported flip does not trigger the new state transition, but all existing penalties continue to apply.

## Motivation

The protocol already withholds validation rewards from identities whose flips are reported, but that penalty may not deter identities that repeatedly publish reportable flips while retaining a validated identity. Repeated reported flips also consume reviewers' attention and reduce the quality of the validation experience.

Exploratory analysis linked below indicates that, in the sampled epochs, a substantial share of reported flips was concentrated among identities with more than one reported flip in the same epoch. The analysis does not establish the author's intent and does not measure the false-positive rate. This proposal therefore uses the protocol's existing report qualification rather than introducing a subjective definition of flip quality, and limits the new penalty to repeated reports in one ceremony.

## Specification

### Definitions

- A **reported flip** is a distinct flip whose final qualification has `grade == GradeReported` under the active consensus rules. Reports discarded by the existing reporter filters do not count. A flip with `QualifiedByNone` status does not count unless its final grade is also `GradeReported`.
- `reportedFlips(author)` is the number of reported flips created by `author` in the validation ceremony being finalized.
- **Pre-validation state** is the identity state at the start of that ceremony, before ordinary validation transitions are calculated.
- **Base next state** is the next identity state produced by all consensus rules that apply without this proposal.

### State transition

For every identity, an implementation MUST first calculate the base next state. It MUST then apply the following rule using the identity's pre-validation state:

| Reported flips | Pre-validation state | Final next state |
| --- | --- | --- |
| 0 or 1 | Any | Base next state |
| 2 or more | `Newbie` | `Killed` |
| 2 or more | `Verified` or `Human` | `Suspended`, unless the base next state is `Killed`; `Killed` MUST be preserved |
| 2 or more | Any other state | Base next state |

The new rule MUST NOT replace an outcome with a less severe state. In particular, a `Verified` identity that fails the ordinary validation rules and would be killed remains `Killed`; it is not changed to `Suspended`.

The final next state MUST be used consistently by all subsequent ceremony-finalization logic, including identity birthday calculation, invitation accounting, validation result construction, and reward processing.

### Existing penalties

This proposal does not change how a reported flip is qualified, how reporters are filtered or rewarded, or how a bad author affects validation and invitation rewards. Those existing rules apply in addition to the state transition above. An identity with exactly one reported flip receives no new state penalty.

### Activation

This rule changes consensus and MUST be disabled before activation. It MUST be enabled by a future consensus upgrade accepted through the existing hard-fork process. The implementation will assign the consensus version and activation window; this proposal does not reserve an application release number, consensus version, fork number, block, or epoch.

## Rationale

The threshold is an absolute count of two rather than a fraction of the author's required flips. This is simple to evaluate and targets repeated failures in one ceremony. A proportional rule such as two of three or three of five was considered in the discussion, but it would leave some identities unpenalized after publishing two reported flips and would add status-dependent thresholds.

The rule uses the pre-validation state so that an identity cannot escape the intended result through an ordinary transition in the same ceremony. It also preserves an ordinary `Killed` result so the new rule can never reduce an existing penalty.

The term `GradeReported` ties the proposal to a value already calculated by consensus. Phrases such as "bad flip," "low effort," or "malicious flip" are motivations, not additional consensus criteria.

## Data illustration

The following chart was generated from Idena API data for the sampled epochs. The red line is the share of all flips attributed to identities with multiple `wrongWords` flips in an epoch; the blue line is the share attributed to identities with exactly one. The bars show total and reported flip volumes.

![Shares of reported flips with volume context per epoch](https://raw.githubusercontent.com/ubiubi18/wrongwordsTRUE/f1e2c36f74f087ceb6d0ecba62625a6b14515017/output%282%29.png)

The [analysis scripts and methodology](https://github.com/ubiubi18/wrongwordsTRUE/tree/f1e2c36f74f087ceb6d0ecba62625a6b14515017) are non-consensus research inputs. The repository does not include the complete raw dataset used for the chart, so the chart should not be treated as independently reproducible evidence or as part of the normative specification.

## Backward compatibility

This change is not backward compatible. Once activated, an old node can calculate a different post-validation identity state and therefore a different state root. Nodes must upgrade before the activation window to remain on the canonical chain.

## Reference implementation

There is no production reference implementation at the time of publication.

An implementation is expected to:

1. Count final `GradeReported` qualifications per author while analyzing flips.
2. Carry the repeated-report result into ceremony finalization without changing existing bad-author reward behavior.
3. Apply the state transition after calculating the base next state and before any consumer records or uses the final next state.
4. Gate the behavior behind the consensus upgrade that activates this proposal.
5. Add integration-level tests for every table row above, including a `Verified` identity whose base next state is already `Killed`, exactly one reported flip, and a `QualifiedByNone` flip that is not `GradeReported`.

The relevant current node paths are [flip qualification](https://github.com/idena-network/idena-go/blob/938be81dbdeff85f888f4337060a8ebabb12e5b5/core/ceremony/qualification.go#L382-L436), [author analysis](https://github.com/idena-network/idena-go/blob/938be81dbdeff85f888f4337060a8ebabb12e5b5/core/ceremony/ceremony.go#L1340-L1412), and [identity-state finalization](https://github.com/idena-network/idena-go/blob/938be81dbdeff85f888f4337060a8ebabb12e5b5/core/ceremony/ceremony.go#L1135-L1174).

## Security considerations

- **False positives:** `GradeReported` is a committee-derived result, not proof of malicious intent. Honest authors can receive two reported grades and would be penalized. Before activation, reviewers should measure the historical false-positive rate and publish reproducible data.
- **Coordinated reporting:** An attacker able to influence enough of a flip's report committee may cause the flip to receive `GradeReported`. This proposal increases the consequence of such influence but does not change the existing report threshold or committee selection.
- **Evasion:** An attacker can avoid the new state penalty by limiting each identity to one reported flip per ceremony or distributing flips across identities. Existing reward penalties still apply, but this proposal does not eliminate that strategy.
- **Network participation:** Suspending or killing repeat offenders can reduce the active validator set, including honest participants caught by false positives. Activation analysis should quantify the expected effect on validator and shard sizes.
- **Determinism:** Nodes must count the same final flip qualifications and apply the transition at the same point in ceremony finalization. Counting raw report transactions, API fields, or reports discarded by existing filters would be consensus-breaking.
- **Penalty monotonicity:** The new transition must never turn an existing `Killed` outcome into `Suspended` or otherwise improve an identity's base next state.

## References

- [Community discussion](https://github.com/idena-network/idena-docs/discussions/187)
- [Historical analysis scripts and chart](https://github.com/ubiubi18/wrongwordsTRUE/tree/f1e2c36f74f087ceb6d0ecba62625a6b14515017)
- [Current `idena-go` ceremony implementation](https://github.com/idena-network/idena-go/tree/938be81dbdeff85f888f4337060a8ebabb12e5b5/core/ceremony)
