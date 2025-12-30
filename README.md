# LogoActions <a href = "https://github.com/danielvartan/logoactions"><img src = "images/logo.svg" align="right" width="120" /></a>

<!-- badges: start -->
[![Project Status: Active – The project has reached a stable, usable state and is being actively developed.](https://www.repostatus.org/badges/latest/active.svg)](https://www.repostatus.org/#active)
[![setup-netlogo](https://github.com/danielvartan/logoactions/actions/workflows/test-setup-netlogo.yaml/badge.svg)](https://github.com/danielvartan/logoactions/actions/workflows/test-setup-netlogo.yaml)
[![check-netlogo](https://github.com/danielvartan/logoactions/actions/workflows/test-check-netlogo.yaml/badge.svg)](https://github.com/danielvartan/logoactions/actions/workflows/test-check-netlogo.yaml)
[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-bd0000.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Contributor Covenant 3.0 Code of Conduct](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://www.contributor-covenant.org/version/3/0/code_of_conduct/)
<!-- badges: end -->

## Overview

`LogoActions` is a collection of [GitHub Actions](https://docs.github.com/en/actions) designed to facilitate the setup and execution of [NetLogo](https://ccl.northwestern.edu/netlogo/) models within [GitHub workflows](https://docs.github.com/en/actions/concepts/workflows-and-actions/workflows). These actions enable researchers and developers to automate the installation of NetLogo, run and test simulations, and integrate NetLogo with other tools and platforms, such as [Quarto](https://quarto.org/), [`logolink`](https://danielvartan.github.io/logolink/) and [`pyNetLogo`](https://pynetlogo.readthedocs.io/en/latest/).

Currently, the repository includes:

- [`setup-netlogo`](#setup-netlogo-reference): Installs NetLogo on the runner machine.
- [`check-netlogo`](#check-netlogo-reference): Runs all BehaviorSpace experiments in a repository's NetLogo models to verify they execute without errors.

Along with these actions, a series of [workflows examples](.github/workflows) are provided to demonstrate how to use the action in practice. See the [Usage](#usage) section below for more details.

> If you find this project useful, please consider giving it a star! &nbsp; [![GitHub Repository Stars](https://img.shields.io/github/stars/danielvartan/logoactions)](https://github.com/danielvartan/logoactions/)

> The continuous development of `LogoActions` depends on community support. If you can afford to do so, please consider becoming a sponsor. &nbsp; [![](https://img.shields.io/static/v1?label=Sponsor&message=%E2%9D%A4&logo=GitHub&color=%23fe8e86)](https://github.com/sponsors/danielvartan)

## Usage

### Check NetLogo Models

This workflow shows how to use the [`check-netlogo`](#check-netlogo-reference) action to automatically verify that all [BehaviorSpace](https://docs.netlogo.org/behaviorspace.html) experiments in a repository's NetLogo models run without errors. This is particularly useful for continuous integration ([CI](https://en.wikipedia.org/wiki/Continuous_integration)) workflows, ensuring that any changes to the models or experiments do not introduce issues.

You can view this workflow in action on the repository's [actions page](https://github.com/danielvartan/logoactions/actions). See the [`LogoClim`](https://github.com/sustentarea/logoclim) model for a real-world example of how this action can catch errors early in the development process.

Below is a basic workflow configuration. To use it, create a file named `check-netlogo.yaml` with the content below and place it in the `.github/workflows` folder at the root of your repository.

```yaml
on:
  push:
    branches: [main, master]

name: NetLogo Check

permissions: read-all

jobs:
  check-netlogo:
    name: Check NetLogo models
    runs-on: ubuntu-latest
    env:
      GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
    steps:
      - name: Check out repository
        uses: actions/checkout@v4

      - name: Set up NetLogo
        uses: danielvartan/logoactions/setup-netlogo@v1

      - name: Check NetLogo models
        uses: danielvartan/logoactions/check-netlogo@v1
```

### Run BehaviorSpace Experiments

This workflow shows how to set up NetLogo and run [BehaviorSpace](https://docs.netlogo.org/behaviorspace.html) experiments in [headless](https://docs.netlogo.org/behaviorspace.html#running-from-the-command-line) mode. Experiment results are saved as [CSV](https://en.wikipedia.org/wiki/Comma-separated_values) files and uploaded as artifacts for later retrieval. This approach is useful for automating simulation runs, conducting parameter sweeps, or integrating NetLogo experiments into data analysis pipelines.

You can view this workflow in action on the repository's [actions page](https://github.com/danielvartan/logoactions/actions). The complete workflow file is available [here](.github/workflows/run-experiment.yaml).

Below is a basic workflow configuration.

```yaml
on:
  push:
    branches: [main, master]

name: Run BehaviorSpace experiments

permissions: read-all

jobs:
  run-experiments:
    name: Run experiments
    runs-on: ubuntu-latest
    env:
      GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
    steps:
      - name: Checkout Repository
        uses: actions/checkout@v4

      - name: Set up NetLogo
        uses: danielvartan/logoactions/setup-netlogo@v1

      - name: Create artifacts directory
        run: |
          # Create artifacts directory

          mkdir -p '/tmp/artifacts'
        shell: bash

      - name: Run experiment
        run: |
          # Run experiment

          model_dir="$NETLOGO_HOME/models/Sample Models/Biology"
          model_file='Wolf Sheep Predation.nlogox'
          experiment_name='Wolf Sheep Crossing'
          table_file='/tmp/artifacts/experiment-table.csv'

          netlogo \
            --headless \
            --model "${model_dir}/${model_file}" \
            --experiment "${experiment_name}" \
            --table "${table_file}"

          cat "${table_file}"
        shell: bash

      - name: Upload artifact
        uses: actions/upload-artifact@v4
        with:
          name: experiment-output
          path: '/tmp/artifacts/'
          retention-days: 90
```

### Run Experiments with Quarto and `logolink`

This workflow shows how to combine NetLogo with [Quarto](https://quarto.org/) and the [`logolink`](https://danielvartan.github.io/logolink/) R package to run experiments and generate reproducible reports. It installs all required dependencies, renders the Quarto document, and deploys the output to [GitHub Pages](https://docs.github.com/en/pages). An example report is available [here](https://danielvartan.github.io/logoactions/).

You can view this workflow in action on the repository's [actions page](https://github.com/danielvartan/logoactions/actions). The complete workflow file is available [here](.github/workflows/run-logolink.yaml).

Below is a basic workflow configuration.

```yaml
on:
  push:
    branches: [main, master]

name: Run experiments with Quarto and logolink

permissions: read-all

jobs:
  run-logolink:
    runs-on: ubuntu-latest
    name: Run experiments
    env:
      GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
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
            libfontconfig1-dev \
            pandoc
        shell: bash

      - name: Set up NetLogo
        uses: danielvartan/logoactions/setup-netlogo@v1

      - name: Set up R
        uses: r-lib/actions/setup-r@v2

      - name: Install and initialize renv
        run: |
          # Install and initialize renv

          install.packages("renv")

          renv::init()
        shell: Rscript {0}

      - name: Set up renv
        uses: r-lib/actions/setup-renv@v2
        with:
          cache-version: 0
          bypass-cache: true

      - name: Set up Quarto
        uses: quarto-dev/quarto-actions/setup@v2

      - name: Render Quarto
        run: |
          # Render Quarto

          quarto render
        shell: bash

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

The `setup-netlogo` action is what allows all the examples shown above to run NetLogo models in GitHub Actions workflows. You can easily customize the action to fit your needs.

### Installation

The action can be integrated into your GitHub Actions workflow simply by adding the following step:

```yaml
- name: Set up NetLogo
  uses: danielvartan/logoactions/setup-netlogo@v1
```

You must also set the `GH_TOKEN` environment variable to enable authenticated GitHub API requests.

```yaml
env:
  GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

This will make NetLogo available for use in subsequent steps.

### Functionality

The action sets up a NetLogo environment by performing the following tasks:

1. Download the specified NetLogo version
2. Cache the installation (optional)
3. Set environment variables:
  - `NETLOGO_HOME`: Installation directory path
  - `NETLOGO_CONSOLE`: Console executable path
  - `NETLOGO_VERSION`: Installed version (e.g., `7.0.3`)
4. Add `NETLOGO_HOME` to `PATH`
5. Create `netlogo` and `NetLogo` symlinks

After the action completes, you can run NetLogo commands in subsequent steps using `netlogo`:

```yaml
- name: Test NetLogo
  run: netlogo --headless --version
```

You can view `setup-netlogo` in action on the repository's [actions page](https://github.com/danielvartan/logoactions/actions).

### Inputs

The following inputs are supported:

- `version`: A single-quoted (!important!) character string indicating the NetLogo version to use (e.g., `'7.0.3'`). Use `'release'` to get the latest
stable release. Only versions 6.4.0 and above are supported (default: `'release'`).
- `cache`: A single-quoted (!important!) boolean value indicating whether the NetLogo installation should be cached across runs (default: `'true'`).

Use the `with` keyword to change the default values. Example:

```yaml
- name: Set up NetLogo
  uses: danielvartan/logoactions/setup-netlogo@v1
  with:
    version: '6.4.0'
    cache: 'false'
```

### Supported Platforms

The runners `ubuntu-latest`, `windows-latest`, and `macos-latest` are supported. We recommend using `ubuntu-latest` whenever possible.

## `check-netlogo` Reference

The `check-netlogo` action automates the verification of [BehaviorSpace](https://docs.netlogo.org/behaviorspace.html) experiments across all NetLogo models in a repository. It ensures experiments run correctly in [headless](https://docs.netlogo.org/behaviorspace.html#running-from-the-command-line) mode, making it ideal for [CI](https://en.wikipedia.org/wiki/Continuous_integration) pipelines that need to catch model errors early.

### Installation

The action can be integrated into your GitHub Actions workflow simply by adding the following step:

```yaml
- name: Check NetLogo models
  uses: danielvartan/logoactions/check-netlogo@v1
```

Before using `check-netlogo`, ensure you [check out](https://github.com/actions/checkout) the repository and set up NetLogo with the [`setup-netlogo`](#setup-netlogo-reference) action.

You must also set the `GH_TOKEN` environment variable to enable authenticated GitHub API requests in `setup-netlogo`.

```yaml
env:
  GH_TOKEN: ${{secrets.GITHUB_TOKEN}}
```

### Functionality

The action test NetLogo models by performing the following tasks:

1. Search the repository for NetLogo model files (`.nlogo`, `.nlogo3d`, `.nlogox`, `.nlogox3d`), excluding any paths specified in the `ignore` input.
2. Identify all BehaviorSpace experiments defined within each model.
3. Execute each experiment in headless mode using NetLogo's command-line interface.
4. Save a table output of each experiment as an artifact (optional).

Note that models with `.nlogo` and `.nlogo3d` extensions are skipped when using NetLogo 7 or higher, as these formats are deprecated in recent versions. Similarly, `.nlogox` and `.nlogox3d` files are skipped when using NetLogo 6 or lower, as these formats are not supported in earlier versions.

You can view `check-netlogo` in action on the repository's [actions page](https://github.com/danielvartan/logoactions/actions).

### Inputs

The following inputs are supported:

- `ignore`: A single-quoted (!important!) character string specifying paths to exclude when searching for NetLogo models. Supports glob patterns (e.g., `'models/old/**'`). Multiple paths can be separated by commas (e.g., `'models/old/**, docs/**'`). No paths are ignored by default (default: `''`).
- `artifacts`: A single-quoted (!important!) boolean value indicating whether to save experiment output tables as artifacts (default: `'true'`).

Use the `with` keyword to change the default values. Example:

```yaml
- name: Check NetLogo models
  uses: danielvartan/logoactions/check-netlogo@v1
  with:
    ignore: 'models/old/**, docs/**'
    artifacts: 'false'
```

### Supported Platforms

This action supports only `ubuntu-latest` runners at the moment.

## Citation

If you use `LogoActions` in your work, please cite it to acknowledge the effort invested in its development and maintenance. Your citation helps support the ongoing improvement of the project.

To cite `LogoActions` in publications please use the following format:

Vartanian, D. (n.d.). *LogoActions: GitHub Actions for the NetLogo community* [Computer software]. <https://github.com/danielvartan/logoactions>

A BibLaTeX entry for LaTeX users is:

```latex
@software{vartanian,
  title = {LogoActions: GitHub Actions for the NetLogo community},
  author = {Daniel Vartanian},
  url = {https://github.com/danielvartan/logoactions},
  note = {Computer software}
}
```

## Contributing

[![Contributor Covenant 3.0 Code of Conduct](https://img.shields.io/badge/Contributor%20Covenant-3.0-4baaaa.svg)](https://www.contributor-covenant.org/version/3/0/code_of_conduct/)

Contributions are always welcome! Whether you want to report bugs, suggest new features, or help improve the code or documentation, your input makes a difference.

Before opening a new issue, please check the [issues tab](https://github.com/danielvartan/logoactions/issues) to see if your topic has already been reported.

## License

[![License: GPLv3](https://img.shields.io/badge/license-GPLv3-bd0000.svg)](https://www.gnu.org/licenses/gpl-3.0)

```text
Copyright (C) 2025 Daniel Vartanian

LogoActions is free software: you can redistribute it and/or modify it
under the terms of the GNU General Public License as published by the Free
Software Foundation, either version 3 of the License, or (at your option) any
later version.

This program is distributed in the hope that it will be useful, but WITHOUT ANY
WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A
PARTICULAR PURPOSE. See the GNU General Public License for more details.

You should have received a copy of the GNU General Public License along with
this program. If not, see <https://www.gnu.org/licenses/>.
```

## Acknowledgments

`LogoActions` is an independent project with no affiliation to [NetLogo](https://www.netlogo.org/) or its developers.

`LogoActions` brand identity is based on the [NetLogo 7](https://www.netlogo.org/) brand identity.
