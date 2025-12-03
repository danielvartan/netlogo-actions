# netlogo-actions

<!-- badges: start -->
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![setup-netlogo](https://github.com/danielvartan/netlogo-actions/actions/workflows/setup-netlogo-test.yaml/badge.svg)](https://github.com/danielvartan/netlogo-actions/actions/workflows/setup-netlogo-test.yaml)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-bd0000.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Contributor Covenant 3.0 Code of Conduct](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://www.contributor-covenant.org/version/3/0/code_of_conduct/)
<!-- badges: end -->

## Overview

`netlogo-actions` is a collection of [GitHub Actions](https://github.com/features/actions) designed to simplify the setup and execution of [NetLogo](https://ccl.northwestern.edu/netlogo/) models within GitHub workflows.

These actions enable researchers and developers to automate NetLogo installation, run simulations, and integrate NetLogo with other tools such as [Quarto](https://quarto.org/) and [`logolink`](https://danielvartan.github.io/logolink/). By using these actions, users can streamline computational modeling workflows, enhance reproducibility, and efficiently manage complex simulations in a CI/CD environment.

Currently, the repository includes only the [`setup-netlogo`](setup/action.yaml) action, which installs NetLogo on the runner machine. See the [`examples`](examples) directory for workflow examples. You can also see `setup-netlogo` in action by checking this repository's [actions](https://github.com/danielvartan/netlogo-actions/actions) page.

This action can also be integrated with [Quarto](https://quarto.org/) using the [`logolink`](https://danielvartan.github.io/logolink/) R package for more advanced workflows. See [`netlogo-check`](https://github.com/danielvartan/netlogo-check) for an example of such integration.

> If you find this project useful, please consider giving it a star! &nbsp; [![GitHub Repository Stars](https://img.shields.io/github/stars/danielvartan/netlogo-actions)](https://github.com/danielvartan/netlogo-actions/)

> The continuous development of `netlogo-actions` depends on community support. If you can afford to do so, please consider becoming a sponsor. &nbsp; [![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/danielvartan)

## `setup-netlogo`

This action sets up a NetLogo environment for use in GitHub Actions by:

- Downloading a specified version of NetLogo.
- Optionally caching the installation across workflow runs.
- Setting the following environment variables:
  - `NETLOGO_HOME`: Path to the NetLogo installation directory.
  - `NETLOGO_CONSOLE`: Path to the NetLogo console executable.
- Adding `NETLOGO_HOME` to `PATH`.
- Creating symlinks for easy access to NetLogo command-line tools:
  - `netlogo`
  - `NetLogo`

### Inputs

The following inputs are supported:

- `version`: The NetLogo version to use (e.g., `"6.4.0"`). See available versions [here](https://ccl.northwestern.edu/netlogo/oldversions.shtml) (default: `"7.0.2"`).
- `architecture`: The NetLogo system architecture to use (`"32"` or `"64"`) (default: `"64"`).
- `cache`: Whether NetLogo should be cached across runs or not (default:`"true"`).

### Basic Usage

A basic workflow to set up NetLogo and run a model might look like this:

```yaml
on:
  push:
    branches: [main, master]

name: "NetLogo Check"

jobs:
  netlogo-check:
    runs-on: ubuntu-latest
    permissions: read-all
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Setup NetLogo
        uses: danielvartan/netlogo-actions/setup-netlogo@v1

      - name: Run Experiment
        run: |
          MODEL_PATH="$NETLOGO_HOME/models/Sample Models/Biology"
          MODEL_FILE="Wolf Sheep Predation.nlogox"

          netlogo \
            --headless \
            --model "$MODEL_PATH/$MODEL_FILE" \
            --experiment "Wolf Sheep Crossing" \
            --table /tmp/output.csv

          cat /tmp/output.csv
```

### Differing Versions

You can also specify a particular NetLogo version to install:

```yaml
steps:
  - name: Checkout Repository
    uses: actions/checkout@v4

  - name: Setup NetLogo 6.4.0
    uses: danielvartan/netlogo-actions/setup-netlogo@v1
    with:
      version: "6.4.0"
      architecture: "64"
```

## Supported Platforms

All actions support only **Linux** runners (e.g., `ubuntu-latest`). We do not plan to support other operating systems at this time.

## License

[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-bd0000.svg)](https://www.gnu.org/licenses/gpl-3.0)

```text
Copyright (C) 2025 Daniel Vartanian

netlogo-actions is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your option) any
later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see <https://www.gnu.org/licenses/>.
```

## Contributing

[![Contributor Covenant 3.0 Code of Conduct](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://www.contributor-covenant.org/version/3/0/code_of_conduct/)

Contributions are always welcome! Whether you want to report bugs, suggest new features, or help improve the code or documentation, your input makes a difference.

Before opening a new issue, please check the [issues tab](https://github.com/danielvartan/netlogo-actions/issues) to see if your topic has already been reported.
