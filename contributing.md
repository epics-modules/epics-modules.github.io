---
layout: default
title: Contributing a Module
nav_order: 2
---

# Contributing a Module

## Adding a Module to the Organization

To add your module to the
[epics-modules](https://github.com/epics-modules) organization:

1. Create an issue in your module's repository mentioning
   `@epics-modules/admins`.
2. The module must include a `LICENSE` file at the top level. The
   [synApps license](https://github.com/epics-modules/busy/blob/master/LICENSE)
   (BSD-style, University of Chicago / UC Regents) is used by most
   modules in the collection, but other open-source licenses are
   acceptable.
3. An active maintainer is expected.

The rest of this page describes conventions followed by many modules
in the collection. They are not requirements, but following them makes
your module easier for others to build, integrate, and contribute to.

---

## Module Directory Structure

Beyond what `makeBaseApp.pl` generates, modules in the collection
typically add the following directories:

```
mymodule/
  .ci/                   ci-scripts submodule (see CI section)
  .github/workflows/     GitHub Actions CI workflow
  configure/             standard EPICS build configuration
  docs/                  GitHub Pages documentation
  iocs/                  separated IOC applications
  mymoduleApp/           module source (Db/, src/)
  Makefile
```

The `*App/` directory name should match the module name
(e.g., `busyApp/` for the busy module).

---

## Build Configuration

### cfg-Based Build Dependencies

IOC application Makefiles must list the DBD files and libraries
provided by each support module they use. When a module adds,
removes, or renames a library or DBD file, every consuming IOC
Makefile must be updated manually. The cfg system lets each module
declare its own build products so that IOC applications can discover
them automatically.

The `CONFIG_MODULE` template handles recursive dependency resolution.
If module A depends on module B, and both have cfg-deps, an IOC that
uses module A automatically discovers module B's build products as
well.

#### Setup

Each module needs five things:

1. `MODULE = <NAME>` in `configure/CONFIG_SITE` (must match the
   variable name used in RELEASE files):

   ```makefile
   MODULE = AUTOSAVE
   ```

2. A `CONFIG_MODULE@` template file in `configure/`. This file is
   identical for every module -- copy it verbatim from any module
   that has it (e.g.,
   [busy](https://github.com/epics-modules/busy/blob/master/configure/CONFIG_MODULE%40)).

3. A declaration file (`<MODULE>_DEPS`) in the module's source
   directory (e.g., `asApp/src/AUTOSAVE_DEPS`).

4. Additions to `configure/Makefile`:

   ```makefile
   CFG += CONFIG_MODULE
   EXPAND += CONFIG_MODULE@
   EXPAND_VARS += MODULE=$(MODULE)
   ```

5. Install the declaration file from the source Makefile
   (e.g., `asApp/src/Makefile`):

   ```makefile
   CFG += AUTOSAVE_DEPS
   ```

#### Declaration Files

A declaration file lists the DBD files and libraries the module
provides to IOC applications.

**Basic** -- a module with a single library and DBD:

```makefile
# asApp/src/AUTOSAVE_DEPS
AUTOSAVE_IOC_DBDS = asSupport.dbd
AUTOSAVE_IOC_LIBS = autosave
```

**Conditional** -- a module with optional components:

```makefile
# sscanApp/src/SSCAN_DEPS
SSCAN_IOC_DBDS = sscanSupport.dbd
SSCAN_IOC_LIBS = sscan

ifdef SNCSEQ
SSCAN_IOC_DBDS += sscanProgressSupport.dbd
SSCAN_IOC_LIBS += scanProgress
endif
```

Declaration files are Make fragments evaluated inside an
`ifdef T_A` guard, so `OS_CLASS`, `T_A`, and `ifdef MODULE`
conditions are all available for platform-specific or
dependency-gated declarations.

#### Modules with Submodules

Modules like motor that contain independently-maintained driver
submodules should not hard-code every submodule's products in a
single declaration file. Instead, the parent declaration file
wildcard-includes fragment files that each submodule installs:

**Parent** (`motorApp/MotorSrc/MOTOR_DEPS`):

```makefile
MOTOR_IOC_DBDS = motorSupport.dbd devSoftMotor.dbd
MOTOR_IOC_LIBS = motor softMotor

# Auto-include all submodule declaration fragments
-include $(_MOTOR_DIR)/MOTOR_*_DEPS
```

**Each submodule** installs a small fragment (e.g.,
`modules/motorNewport/newportApp/src/MOTOR_NEWPORT_DEPS`):

```makefile
MOTOR_IOC_DBDS += devNewport.dbd
MOTOR_IOC_LIBS += Newport
```

The submodule's Makefile just needs `CFG += MOTOR_NEWPORT_DEPS`.

When submodules are added or removed, no central file needs
updating -- the wildcard automatically discovers whatever
fragments are installed. Submodule fragments can use the same
platform and dependency guards as regular declaration files.

#### How IOCs Consume

IOC Makefiles use per-module variables to select which modules
to include:

```makefile
$(DBD_NAME)_DBD += $(AUTOSAVE_IOC_DBDS)
$(DBD_NAME)_DBD += $(MOTOR_IOC_DBDS)

$(PROD_NAME)_LIBS += $(MOTOR_IOC_LIBS)
$(PROD_NAME)_LIBS += $(AUTOSAVE_IOC_LIBS)
$(PROD_NAME)_LIBS += $(EPICS_BASE_IOC_LIBS)
```

---

## Continuous Integration

Most modules use [epics-base/ci-scripts](https://github.com/epics-base/ci-scripts)
as a `.ci/` git submodule, with a GitHub Actions workflow in
`.github/workflows/`.

The test matrix should cover at minimum:

- Linux (gcc)
- macOS (clang)
- Windows (MSVC, static build)

Modules should be tested against both stable releases and master
branches of their dependencies. This catches integration issues
early and validates that cfg-deps declarations stay current.

A typical workflow structure:

```yaml
on:
  push:
    branches: [master]
  pull_request:

env:
  SETUP_PATH: .github/workflows:.ci

jobs:
  build:
    strategy:
      matrix:
        include:
          - os: ubuntu-latest
            set: linux-stable

          - os: ubuntu-latest
            set: linux-master

          - os: macos-latest
            set: macos

          - os: windows-latest
            set: windows

    runs-on: ${{ matrix.os }}
    steps:
      - uses: actions/checkout@v4
        with:
          submodules: true
      - run: python .ci/cue.py prepare
      - run: python .ci/cue.py build
      - run: python .ci/cue.py test
```

The `SET` files (e.g., `linux-stable.set`, `linux-master.set`) in
`.github/workflows/` define which dependency versions to build
against.

---

## Releases

- Tag releases as `R<major>-<minor>[-<patch>]` (e.g., `R1-7-4`).
- Create GitHub Releases from tags.
- Maintain release notes in `docs/` (e.g.,
  `docs/<module>ReleaseNotes.md`).
