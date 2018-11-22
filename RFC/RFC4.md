# pgRouting RFC 4: Handling releases and branches on pgRoutingLayer

| Item | value
| --- | ---
| RFC 4 | Handling releases and branches on pgRoutingLayer
| Author(s) | Cayetano Benavent
| Status | waiting for feedback
| date | November 18, 2018

## Table of Contents

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

* Handling releases and branches for the [pgRoutingLayer](https://github.com/pgRouting/pgRoutingLayer) repository.

This document is based on [pgRouting RFC 2: Handling releases and branches on pgRouting](RFC2.md)

## Details

### Naming

#### Tags

* Tag release names: `v<major>.<minor>.<patch>[-label]`

Examples:

* [`v2.2.2`](https://github.com/pgRouting/pgRoutingLayer/releases/tag/v2.2.2)
* [`v3.0.0-alpha1`](https://github.com/pgRouting/pgRoutingLayer/releases/tag/v3.0.0-alpha1)
* `v3.0.0-dev` (no tag is created)

References:

* [Semantic versioning](https://semver.org/)
* [creating releases](https://help.github.com/articles/creating-releases/)
* [tagging basics](https://git-scm.com/book/en/v2/Git-Basics-Tagging)
* [git tag](https://git-scm.com/docs/git-tag)
* [git verify-tag](https://git-scm.com/docs/git-verify-tag)

#### Naming conventions

* _Current release_: `v<M>.<m>.<p>`
* _Next Major_: `v<M + 1>.0.0`
* _Next minor_: `v<M>.<m + 1>.<0>`
* _Next Patch_: `v<M>.<m>.<p + 1>`

### Patch releases

* Purpose:
  * Fix code & security bugs related to QGIS Python API (PyQGIS or PyQt).
  * Fix code & security bugs related to PostgreSQL/PostGIS connections.
  * Fix code & security bugs related to pgRouting functions available in pgRoutingLayer.
* Commits: **In one commit**
* Do in:
  * _Current release_ `v<M>.<m>.<p>`
    * Create a Next patch`v<M>.<m>.<p + 1>`
  * Back port to:
    * For all the Minors of the current Major `v<M>.<m>`
      * Create a next patch `v<M>.<m>.<p + 1>`
    * For the previous Major `v<M>` Where `M != 1` and only to the last minor `<m>`:
      * Create a next patch `v<M>.<m>.<p + 1>`
* Create tags:
  * starting from the lowest version up to the _current release_ (because of the way GitHub marks the latest release)
  * **On the same day all tags that apply**.
* Do not need `-alpha` `-beta` `-rc` labels

### Minor releases

* Purpose:
  * Minor changes at QGIS Python API.
  * Add new functions from pgRouting stable version.
  * Remove deprecated functions following pgRouting deprecations on stable version.
* Commits: As many as needed

### Major releases

* Purpose:
  * Major changes (API break) at QGIS Python API (i.e. Migration from QGIS 2 to QGIS 3).
  * Upgrade to pgRouting major release.
* Commits: As many as needed
* No back porting is performed.
* create an `-alpha`, `-beta`, `-rc` releases (as many as needed) before the final Major release is done.

## Releases plan

* pgRoutingLayer releases plan is closely related to [pgRouting releases plan](RFC2.md).


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
