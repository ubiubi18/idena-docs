---
hide_title: true
title: IIP-14
sidebar_label: IIP-14 Repeated Misconduct Suspension
---

## IIP-14: Suspension of Identities who publish Multiple Bad Flips per Epoch

`Author`: ubiubi18  
`Status`: Draft  
`Type`: Standard  
`Created`: 2025-08-05  
`Discussion`: https://github.com/idena-network/idena-docs/discussions/187

`Translations`:

## Abstract

This proposal introduces a new protocol rule: **Any identity creating two or more reported (“bad”) flips within the same validation epoch will be suspended (if Verified/Human) or killed (if Newbie) after the ceremony.** Identities with only one reported flip in a session are not affected. This is to deter repeated low-effort or malicious flip creation, which undermines protocol security and user experience.

## Motivation

The share of reported bad flips in Idena ceremonies has increased significantly over the past year, exceeding 16% of all flips per epoch. Data shows only 3-5% of reported flips are likely published by honest users (one-off, unintentional errors). The majority are produced by identities publishing multiple bad flips in one session - suggesting intent to abuse protocol rules. These actors retain identity and mining rights, harming the protocol and the experience of honest users. The current penalty (loss of session rewards) is insufficient and unfair, especially when compared to accidental single mistakes. More bad flips also increase the likelihood that Idena will fail to impress first-time users at first glance. Eliminating almost all bad flips will undoubtedly improve Idena’s reputation and open up new perspectives for the network.

**Strengthening the penalty for repeated bad flips will discourage abuse, improve overall flip quality, and protect protocol security.**

## Specification

During each validation ceremony:

- **Count** the number of flips reported as bad (majority-reported) for every identity.

If an identity creates **two or more bad flips in the same session**:

- If status is **Newbie**: Mark identity as **Killed** at validation end (full stake burnt, mining/voting rights revoked).
- If status is **Verified or Human**: Mark identity as **Suspended** at validation end (identity loses mining/voting rights until next successful validation).

Identities with only one reported flip in a session continue under existing penalty (loss of session rewards).

Penalties are applied **automatically after validation**.

**Protocol version:** 1.2.0 (Fork 14).  
**Activation:** Requires hard fork.

## Rationale

Analysis of Idena validation data (see below) shows **most bad flips come from a small group of addresses** publishing multiple bad flips per epoch. At current network size, just ~40 to ~50 such identities per epoch can degrade protocol trust and user experience while suffering only minor penalties. In contrast, honest users making a single mistake are punished almost equally. This change introduces a strong deterrent: repeat offenders are immediately suspended or killed, protecting the network and rewarding high-quality flip creation.

Alternative ideas (lower penalties, weighted reporting, penalties only for addresses with 3 reported flips per epoch) were considered in [community discussion](https://github.com/idena-network/idena-docs/discussions/187). But the data reflect a different reality and it seems as less strict rules would not prevent serial abuse or would unfairly punish honest users. **This approach targets only repeat offenders, minimizing collateral damage.**

There are good reasons to assume that identities with more than one reported flip per epoch do so intentionally, while single reported flips are most often honest mistakes - which are rare, as flip creation takes considerable effort.

## Data Illustration

The chart below summarizes the historic share of reported bad flips in Idena ceremonies, breaking down the share attributed to addresses with multiple reports per epoch (red), single reports (blue), and the context of overall flip volumes:

![Shares of Reported Bad Flips with Volume Context per Epoch](https://github.com/ubiubi18/wrongwordsTRUE/blob/main/output(2).png)

*See repo for code and raw data: https://github.com/ubiubi18/wrongwordsTRUE/tree/main*

- The **red line** shows the share of flips by addresses with multiple reports per epoch - the key target of this proposal.
- The **blue line** represents the share by addresses with only one report (mostly honest mistakes).
- **Gray bars**: total flips per epoch; **red bars**: flips reported as bad.

## Backward Compatibility

This proposal **requires a hard fork** (protocol upgrade) at a specific block/epoch.  
Nodes running old software will be unable to validate or mine after the fork block.

## Reference Implementation

*requires help, early vibe-coded attempt, see: [Idena-Go.Hard.Fork.Implementation.Consensus.V13.Reported.Flips.Penalty(1).pdf](https://github.com/ubiubi18/wrongwordsTRUE/blob/main/Idena-Go.Hard.Fork.Implementation.Consensus.V13.Reported.Flips.Penalty(1).pdf)*

## Security Considerations

- **Prevents protocol flooding:** Attackers can no longer retain identities while flooding the protocol with bad flips; quality is enforced.
- **Reduces collusion risk:** Majority-reported flips make it statistically hard for honest users to be wrongly penalized.
- **Minimizes accidental harm:** Almost all one-off mistakes result in only minor penalties; only serial abusers are suspended/killed.
- **Attackers must restart from scratch:** Killed identities require re-invitation/validation, making repeated attacks costly and time-consuming.
- **No increased risk for honest users:** Users with only one reported flip per session are not subject to new penalties.
- **Network-size might shrink after implementation:** Some resistance from bad players to the IIP-implementation is to be expected; around 8% of all Idena identities are expected to be suspended under the IIP proposal.

## References

- [Discussion: IIP-14 — Suspend identities with multiple bad flips per epoch](https://github.com/idena-network/idena-docs/discussions/187)
- [Historical flip report dataset and scripts to collect data from public api](https://github.com/ubiubi18/wrongwordsTRUE/tree/main)

