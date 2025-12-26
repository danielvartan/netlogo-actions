# v1.2.1.9000 (development version)

- `[setup-netlogo]` had the `architecture` argument removed. The action now automatically detects the architecture of the runner machine.
- `[setup-netlogo]` now supports macOS and Windows runners in addition to Linux runners.
- `[check-netlogo`] now check OS compatibility before running Behaviorspace experiments.

# v1.2.1 (2025-12-04)

- `[check-netlogo]` now suppresses harmless warnings related to [mediaLib](https://en.wikipedia.org/wiki/MediaLib). These warnings occur when NetLogo attempts to load a GUI-related library in headless mode while using the [GIS extension](https://github.com/NetLogo/GIS-Extension). See [this issue](https://github.com/NetLogo/GIS-Extension/issues/4) for more details.

# v1.2.0 (2025-12-03)

- `[check-netlogo]` action was added. This action runs all Behaviorspace experiments define in NetLogo models present in a repository and uploads the results as artifacts.

- `NetLogo-Actions` now have a [hex logo](images/logo.svg).

# v1.1.0 (2025-12-03)

- `[setup-netlogo]` now supports `'release'` as input for the `version` argument, which downloads the latest NetLogo release.
- `[setup-netlogo]` now downloads NetLogo from the [NetLogo GitHub releases](https://github.com/NetLogo/NetLogo/releases) instead of [ccl.northwestern.edu](https://ccl.northwestern.edu/netlogo/oldversions.shtml), improving reliability and speed.
- `[setup-netlogo]` now supports only NetLogo versions 6.4.0 and above. This limitation is due to the fact that older versions are not available on the NetLogo GitHub releases page.
- `[setup-netlogo]` now follows the [Google Shell Style Guide](https://google.github.io/styleguide/shellguide.html).
- `[setup-netlogo]` now has more informative and less verbose log output.
- `[setup-netlogo]` now exports a `NETLOGO_VERSION` environment variable.
- `[setup-netlogo]` now includes argument checks and error handling.
- `[setup-netlogo]` now tests NetLogo with `--headless --version` instead of `--help`, avoiding possible errors regarding missing GUI dependencies.

- The workflow examples were improved to reflect the changes above.
- The documentation was updated to reflect the changes above.

# v1.0.0 (2025-12-01)

- First release! 🎉

# v0.0.0.9000

- Added a `NEWS.md` file to track changes.
