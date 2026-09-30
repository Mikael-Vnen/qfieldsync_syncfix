# Issue

On Windows, QFieldSync fails to apply downloaded QFieldCloud changes to the currently open QGIS project after a GeoPackage layer has been edited.

The cloud download succeeds, but QGIS keeps an open OGR/SQLite handle to the edited .gpkg file. QFieldSync then fails to replace that file in the project directory because Windows does not allow the active GeoPackage to be overwritten while the file handle is active.

This results in error message: 
CRITICAL Failed to copy project files to the project directory!

The downloaded GeoPackage is up to date, but the active project continues using the stale local copy.

# Fix

Before overwriting the GeoPackages QFieldSync now:

Stores the current QGIS project filename.

Clears the current QgsProject to release active layer/provider handles.

Waits briefly on Windows to allow the GeoPackage file handles to be fully released.

Copies the downloaded files into the project directory.

Reopens the original QGIS project.

The current-project detection was also changed to compare the normalized directory of the loaded project file against the QFieldCloud project's local directory. This proved more reliable than the existing is_current_qgis_project check based on QgsProject.homePath().

File(s)
qfieldsync/core/cloud_transferrer.py

# Changed lines:

Imports
25, 26, 28, 38

New functions
353 _on_download_finished()
503 _import_downloaded_poject()


Result

After the change, GeoPackage updates downloaded from QFieldCloud can be applied successfully even when the affected layers have previously been edited in the current QGIS session. Bidirectional synchronization works without requiring the user to close QGIS or manually re-download the project.




# \/ Original QFieldSync readme text \/

[![Read the documentation](https://img.shields.io/badge/Read-the%20docs-green.svg)](https://docs.qfield.org/get-started/)
[![Release](https://img.shields.io/github/release/opengisch/QFieldSync.svg)](https://github.com/opengisch/QFieldSync/releases)
[![Build Status](https://travis-ci.org/opengisch/qfieldsync.svg?branch=master)](https://travis-ci.org/opengisch/qfieldsync)

# QFieldSync

This plugin facilitates packaging and synchronizing QGIS projects for use with [QField](http://www.qfield.org).

It analyses the QGIS project and suggests and performs actions needed to make the project working on QField.

More information can be found in the [QField documentation](https://docs.qfield.org/get-started/).

The plugin can be download on the [QGIS plugin repository](https://plugins.qgis.org/plugins/qfieldsync/).


## Contribute

QFieldSync is an open source project, licensed under the terms of the GPLv3 or later.
This means that it is free to use and modify and will stay like that.

We are very happy if this app helps you to get your job done or in whatever creative way you may use it.

If you found it useful, we will be even happier if you could give something back.
A couple of things you can do are:

- Rate the plugin at [plugins.qgis.org](https://plugins.qgis.org/plugins/qfieldsync/) ★★★★★
- Write about your experience (please let us know!).
- [Help with the documentation](https://github.com/opengisch/QField-docs/).
- Translate [the QFieldSync QGIS plugin](https://app.transifex.com/opengisch/qfieldsync/dashboard/), [the QField app](https://app.transifex.com/opengisch/qfield-for-qgis/dashboard/) or [the documentation](https://app.transifex.com/opengisch/qfield-documentation/dashboard/).
- [Sponsor a feature](https://docs.qfield.org/get-started/sponsor/)
- And just drop by to say thank you or have a beer with us next time you meet [OPENGIS.ch](https://opengis.ch) at a conference.
- [Develop a new feature or fix a bug](#development).


## Development

### Getting the source code

1) Checkout [`qfieldsync`](https://github.com/opengisch/qfieldsync/) locally:

```shell
git clone --recurse-submodules git@github.com:opengisch/qfieldsync.git
```

2) Make a link of the QFieldSync checkout to `qfieldsync` directory in your current QGIS profile:

```shell
ln -s ${PWD}/qfieldsync/qfieldsync ${HOME}/.local/share/QGIS/QGIS3/profiles/default/python/plugins
```

3) Checkout [`libqfieldsync`](https://github.com/opengisch/libqfieldsync/) locally:

```shell
git clone git@github.com:opengisch/libqfieldsync.git
```

4) Install your local `libqfieldsync` as editable dependency (assuming you are in the same directory as step 3):

```shell
pip install -e libqfieldsync
```

> [!NOTE]
> On more recent Linux distributions you might get an error `error: externally-managed-environment` and you have to pass additional `--break-system-packages`.
> Despite the name, we promise this is not going to break system packages.

> [!NOTE]
> On **macOS**, QGIS bundles its own Python interpreter. Use it instead of the system `pip`:
>
> ```shell
> /Applications/QGIS.app/Contents/MacOS/python -m pip install -e ./libqfieldsync
> ```
>

### Opening a PR

Make sure each new feature or bug fix are in a separate PR.

QFieldSync stores the respective `libqfieldsync` commit SHA in the bottom of [`requirements.txt`](https://github.com/opengisch/qfieldsync/blob/master/requirements.txt#L9-L10).
Sometimes changes in QFieldSync require modifications in [`libqfieldsync`](https://github.com/opengisch/libqfieldsync/).
In these cases please update the commit sha of `libqfieldsync` to point to the respective commit on `libqfieldsync`'s master branch.


## Testing

Run local tests (assuming a QGIS installed on host):

```shell
uv run pytest
```

If you want to test with a specific QGIS version, or you don't have QGIS installed, then:

```shell
docker run --rm $(docker build --build-arg QGIS_TEST_VERSION=ltr -q -f .docker/Dockerfile .) .docker/xvfb-pytest
```

> [!NOTE]
> On Apple Silicon add `--platform linux/amd64`. `qgis/qgis` image is only published for `linux/amd64`.
>
> ```shell
> docker run --rm $(docker build --platform linux/amd64 --build-arg QGIS_TEST_VERSION=ltr -q -f .docker/Dockerfile .) .docker/xvfb-pytest
> ```
