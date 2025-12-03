# NetLogo-Actions <a href = "https://danielvartan.github.io/brandr/"><img src = "images/logo.svg" align="right" width="120" /></a>

<!-- badges: start -->
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![setup-netlogo](https://github.com/danielvartan/netlogo-actions/actions/workflows/test-setup-netlogo.yaml/badge.svg)](https://github.com/danielvartan/netlogo-actions/actions/workflows/test-setup-netlogo.yaml)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-bd0000.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Contributor Covenant 3.0 Code of Conduct](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://www.contributor-covenant.org/version/3/0/code_of_conduct/)
<!-- badges: end -->

## Overview

`NetLogo-Actions` is a collection of [GitHub Actions](https://docs.github.com/en/actions) designed to facilitate the setup and execution of [NetLogo](https://ccl.northwestern.edu/netlogo/) models within [GitHub workflows](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows). These actions enable researchers and developers to automate the installation of NetLogo, run and test simulations, and integrate NetLogo with other tools and platforms, such as [Quarto](https://quarto.org/), [`logolink`](https://danielvartan.github.io/logolink/) and [`pyNetLogo`](https://pynetlogo.readthedocs.io/en/latest/).

Currently, the repository includes only the [`setup-netlogo`](setup/action.yaml) action, which installs NetLogo on the runner machine.

Along with `setup-netlogo`, a series of [workflows examples](.github/workflows) are provided to demonstrate how to use the action in practice. See the [Usage](#usage) section below for more details.

> If you find this project useful, please consider giving it a star! &nbsp; [![GitHub Repository Stars](https://img.shields.io/github/stars/danielvartan/netlogo-actions)](https://github.com/danielvartan/netlogo-actions/)

> The continuous development of `NetLogo-Actions` depends on community support. If you can afford to do so, please consider becoming a sponsor. &nbsp; [![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/danielvartan)

## Usage

### Running BehaviorSpace Experiments

This workflow shows how to set up NetLogo and run [BehaviorSpace](https://docs.netlogo.org/behaviorspace.html) experiments in headless mode. Experiment results are saved as [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) files and uploaded as artifacts for later retrieval. This approach is useful for automating simulation runs, conducting parameter sweeps, or integrating NetLogo experiments into data analysis pipelines.

You can view this workflow in action on the repository's [actions page](https://github.com/danielvartan/netlogo-actions/actions). The complete workflow file is available [here](.github/workflows/run-experiment.yaml).

Below is a sample workflow configuration.

```yaml
on:
  push:
    branches: [main, master]

name: "Run BehaviorSpace Experiments"

permissions: read-all

jobs:
  run-experiments:
    runs-on: ubuntu-latest
    name: Run Experiments
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up NetLogo
        uses: danielvartan/netlogo-actions/setup-netlogo@v1

      - name: Create artifacts directory
        run: |
          # Create artifacts directory
          mkdir -p /tmp/artifacts

      - name: Run experiment
        run: |
          # Run experiment
          model_dir="$NETLOGO_HOME/models/Sample Models/Biology"
          model_file="Wolf Sheep Predation.nlogox"
          experiment_name="Wolf Sheep Crossing"
          table_file="/tmp/artifacts/experiment-table.csv"

          netlogo \
            --headless \
            --model "${model_dir}/${model_file}" \
            --experiment "${experiment_name}" \
            --table "${table_file}"

          cat "${table_file}"

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: experiment-output
          path: /tmp/artifacts/
          retention-days: 90
```

### Running Experiments with Quarto and `logolink`

This workflow shows how to combine NetLogo with [Quarto](https://quarto.org/) and the [`logolink`](https://danielvartan.github.io/logolink/) R package to run experiments and generate reproducible reports. It installs all required dependencies, renders the Quarto document, and deploys the output to [GitHub Pages](https://docs.github.com/en/pages). An example report is available [here](https://danielvartan.github.io/netlogo-actions/).

You can view this workflow in action on the repository's [actions page](https://github.com/danielvartan/netlogo-actions/actions). The complete workflow file is available [here](.github/workflows/run-logolink.yaml).

Below is a sample workflow configuration.

```yaml
on:
  push:
    branches: [main, master]

name: "Run Experiments with Quarto and logolink"

jobs:
  run-logolink:
    runs-on: ubuntu-latest
    name: Run Experiments
    permissions:
      contents: write
      pages: write
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Install dependencies
        run: |
          # Install dependencies
          sudo apt-get update -qq
          sudo apt-get install -y -qq \
            libfontconfig1-dev pandoc

      - name: Set up NetLogo
        uses: danielvartan/netlogo-actions/setup-netlogo@v1

      - name: Set up R
        uses: r-lib/actions/setup-r@v2

      - name: Check if renv is initialized
        id: renv-check
        run: |
          # Check if renv is initialized
          if [ -f "renv.lock" ]; then
            echo "exists=true" >> $GITHUB_OUTPUT
          else
            echo "exists=false" >> $GITHUB_OUTPUT
          fi

      - name: Install and initialize renv
        if: steps.renv-check.outputs.exists == 'false'
        run: |
          # Install and initialize renv
          install.packages("renv")
          renv::init()
        shell: Rscript {0}

      - name: Install logolink
        if: steps.renv-check.outputs.exists == 'false'
        run: |
          # Install logolink
          renv::install("danielvartan/logolink")
          renv::snapshot()
        shell: Rscript {0}

      - name: Set up renv
        uses: r-lib/actions/setup-renv@v2

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2

      - name: Render Quarto
        run: |
          # Render Quarto
          quarto render

      - name: Deploy to GitHub Pages
        if: github.event_name != 'pull_request'
        uses: JamesIves/github-pages-deploy-action@v4.5.0
        with:
          clean: false
          branch: gh-pages
          folder: docs
```

For information on workflow RAM, storage, and time limits, refer to the GitHub Actions [usage limits](https://docs.github.com/en/actions/administering-github-actions/usage-limits-billing-and-administration) page.

## `setup-netlogo` Reference

The `setup-netlogo` action is what allows the examples shown above to run NetLogo models in GitHub Actions workflows. You can easily customize the action to fit your needs.

### Installation

The action can be integrated into your GitHub Actions workflow simply by adding the following step:

```yaml
- name: Set up NetLogo
  uses: danielvartan/netlogo-actions/setup-netlogo@v1
```

This will make NetLogo available for use in subsequent steps of your workflow.

The action sets up a NetLogo environment by performing the following tasks:

1. Downloads the specified NetLogo version
2. Caches the installation (optional)
3. Sets environment variables:
  - `NETLOGO_HOME`: Installation directory path
  - `NETLOGO_CONSOLE`: Console executable path
  - `NETLOGO_VERSION`: Installed version
4. Adds `NETLOGO_HOME` to `PATH`
5. Creates `netlogo` and `NetLogo` symlinks

After the action completes, you can run NetLogo commands in subsequent steps using the `netlogo` command:

```yaml
- name: Test NetLogo
  run: netlogo --headless --version
```

You can view `setup-netlogo` in action on the repository's [actions page](https://github.com/danielvartan/netlogo-actions/actions).

### Inputs

The following inputs are supported:

- `version`: A single-quoted (!important) character string indicating the NetLogo version to use (e.g., `'7.0.2'`). Use `'release'` to get the latest
stable release. Only versions 6.4.0 and above are supported (default: `'release'`).
- `architecture`: A single-quoted (!important) character string indicating the NetLogo system architecture to use. Options are `'32'` or `'64'` (default: `'64'`).
- `cache`: A single-quoted (!important) boolean value indicating whether the NetLogo installation should be cached across runs (default: `'true'`).

Use the `with` keyword to change the default values. Example:

```yaml
- name: Set up NetLogo
  uses: danielvartan/netlogo-actions/setup-netlogo@v1
  with:
    version: '6.4.0'
    architecture: '64'
    cache: 'true'
```

## Supported Platforms

All actions support only **Linux** runners (e.g., `ubuntu-latest`). I do not plan to support other operating systems.

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
