# pgRouting RFC 2: Handling releases and branches on pgRouting

| Item | value
| --- | ---
| RFC 2 | Handling releases and branches on pgRouting
| Author(s) | Vicky Vergara
| Status | Adopted
| date | October 24, 2018

Table of Contents
=================

* [pgRouting RFC 2: Handling releases and branches on pgRouting](#pgrouting-rfc-3-handling-releases-and-branches-on-pgrouting)
   * [Summary](#summary)
   * [Details](#details)
      * [Naming](#naming)
      * [Patch releases](#patch-releases)
      * [Minor releases](#minor-releases)
      * [Major releases](#major-releases)
   * [Releases plan](#releases-plan)
      * [Proposed dates:](#proposed-dates)
   * [Branches](#branches)
      * [Main repository branches](#main-repository-branches)
      * [GSoC Repository &amp; Branches](#gsoc-repository--branches)

## Summary
This document proposes how to proceed on:
- Handling releases and branches for the [pgRouting](https://github.com/pgRouting/pgrouting) repository.

As we move forward to version 3.0, and internally is reaching a more stable state.


## Details

### Naming

**Tags**
- Tag release names: `v<major>.<minor>.<patch>[-label]` 

Examples:
  - [v2.6.1](https://github.com/pgRouting/pgrouting/releases/tag/v2.6.1)
  - [v2.5.0-rc](https://github.com/pgRouting/pgrouting/releases/tag/v2.5.0-rc)
  - v3.0.0-dev (no tag is created)

References
- [Semantic versioning](https://semver.org/)
- [creating releases](https://help.github.com/articles/creating-releases/)
- [tagging basics](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
- [git tag](https://git-scm.com/docs/git-tag)
- [git verify-tag](https://git-scm.com/docs/git-verify-tag)

**Naming conventions**
- _Current release_: `v<M>.<m>.<p>`
- _Next Major_: `v<M + 1>.0.0`
- _Next minor_: `v<M>.<m + 1>.<0>`
- _Next Patch_: `v<M>.<m>.<p + 1>`

### Patch releases
- Purpose:
  - Fix code & security bugs in **official** functions.
  - Fix security bugs in _experimental_ or _proposed_ functions (like server crashes)
- commits: **In one commit**
- Do in:
  - _Current release_ `v<M>.<m>.<p>`
    - Create a Next patch`v<M>.<m>.<p + 1>`
  - Back port to:
    - For all the Minors of the current Major v<M>.<m>
       - Create a next patch `v<M>.<m>.<p + 1>`
    - For the previous Major `v<M>` Where `M != 1` and only to the last minor <m>:
       - Create a next patch `v<M>.<m>.<p + 1>`
- Create tags:
  - starting from the lowest version up to the _current release_ (because of the way GitHub marks the latest release)
  - **On the same day all tags that apply**.
- Do not need `-alpha` `-beta` `-rc` labels


**Example**: if v4.2.1 has a "non security bug" in an official function `pgr_foo` in v4.2 but not official in v.3.3
- fix is on `v4.2.2` with commit number `xxxxxxxx`
- fix micros of current release:
  - back port to `v4.1.p` creating `v4.1.p+1` (if it has the same bug)
  - back port to `v4.0.p` creating `v4.0.p+1` (if it has the same bug)
- Fix previous Major:
  - back port **is not done** to `v3.3.p` because `pgr_foo` **is not official** in this version
    - do the back port if the bug fix is for a security bug
    - do the back port if the function is **official**
  - back port **is not done** to `v3.2.p` because it is not the last minor
  - back port **is not done** to: `v2.6.p` because is not the last Major
- tags are created in this order: `v3.3.p+1` `v4.0.p+1` `v4.1.p+1` being the last tag to create `v4.2.2`



**Back Porting**

if the code has the same bug, this applies:
- Create a testing branch and test the patch
```
git checkout v3.3.1
git checkout -b test-v3.3.1
git cherry-pick xxxxxxxx
```
To back port:
- Create (if not exists) branch release/3.3 on main repository
- Pump up the version (if not pumped yet) from `v3.3.p` to `v3.3.p+1`
- apply the patch
  

References
- [git cherry-pick](https://git-scm.com/docs/git-cherry-pick)


### Minor releases
- Purpose:
  - Add experimental functions.
  - Add proposed functions.
  - Deprecate functions.
  - Move up from experimental to proposed functions
    - Normally because a code changes that include:
      - fixing bugs of experimental or proposed functions
      - Adding tests
- Commits: As many as needed
- No back porting is performed.
- **No official functions added**
- create an `-rc` release before the finial minor release is done

### Major releases
- purpose
  - Move up experimental functions to proposed functions 
  - Move up proposed functions to official functions
  - Remove deprecated functions, add them to legacy file(s)
- Commits: As many as needed
- No back porting is performed.
- create an `-alpha`, `-beta`, `-rc` releases (as many as needed) before the final Major release is done

## Releases plan

- Every 2 years have a **Major** release
  - _Proposed_ functions moves up to **official**
  - _experimental_ functions moves up to **official**
  - _Proposed_ functions moves down to _experimental_
  - _experimental_ functions moves up to _Proposed_
  - Remove deprecated functions
- Every 6 months (March, September, March) within the 2 years have a **Minor** release
  - _experimental_ functions moves up to _Proposed_
  - _Proposed_ functions moves down to _experimental_
  - add new _experimental_ or _Proposed_ functions
  - Deprecate functions
- March releases:
  - Adds GSoC previous year functions as _experimental_ depending on quality
    - This gives 6 months to students to improve quality
- Once a Major release is out, support for previous release is done on the last minor-patch
- Supporting releases is done only to the last 2 Majors, only in the last micro-patch for bug fixes on the official functions

### Proposed dates:

| Release | kind | release date | patch support ends
| ---- | ------ | ------- | ----------
| v 2.6.1 | minor | - | September 2021
| v3.0.0 | Major | September 2019 | September 2021
| v3.1.0 | Minor | March 2020 | September 2021
| v3.2.0 | Minor | September 2020 | September 2021
| v3.3.0 | Last Minor of v3| March 2021 | September 2023
| v4.0.0 | Major | September 2021 | No longer patch support to Major v2.<last minor> on official functions
| v5.0.0 | Major | September 2023 | No longer patch support to Major v3.<last minor> on official functions

| Version | release | patch support on official functions ends
| ---- | ------ | -------
| 2 | September 2013 | September 2021
| 3 | September 2019 | September 2023
| 4 | September 2021 | September 2025
| 5 | September 2023 | September 2027


## Branches

### Main repository branches

| branch | contains
| -------|---------|
| master | The next patch release
| develop | The next release (minor or major)
| gh-pages | documentation
| `release/v<M>.<m>` | Temporary (could be permanent while supported?) branches created for patch releases

### GSoC Repository & Branches

Normally during GSoc the main repository gets full of branches for students to use.

Have a **pgrouting-GSoC** repository where the students can make their PR to.
That repository would work also for saving their tags once the program is finished
