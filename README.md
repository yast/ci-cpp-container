# The YaST C++ Testing Image

[![CI](https://github.com/yast/ci-cpp-container/actions/workflows/ci.yml/badge.svg?branch=master)](https://github.com/yast/ci-cpp-container/actions/workflows/ci.yml)

This git repository contains the configuration used to build the docker
image used for [TravisCI](https://travis-ci.org/).
The resulting docker image is available at https://registry.opensuse.org/.

## Automatic Rebuilds

- The image is rebuilt whenever a commit it pushed to the `master` branch.
- The [yast-ci-cpp-container-master](
  https://ci.opensuse.org/view/Yast/job/yast-ci-cpp-container-master/)
  Jenkins job copies the configuration to the [YaST:Head/ci-cpp-container](
  https://build.opensuse.org/package/show/YaST:Head/ci-cpp-container)
  OBS project
- The OBS tracks the dependencies and rebuilds the image if any dependant package
  is updated.

## Triggering a Rebuild Manually

If for some reason the automatic rebuild do not work or it failed you can
trigger the rebuild in the OBS just like for the other regular packages.


## The Image Content

This image is based on the latest openSUSE Tumbleweed image, additionally
it contains the packages needed for running the tests for YaST packages written
in C++. It is possible to install additional packagers if needed, see the
[Examples](#examples) section below.

## Using the Image in the Other Projects

The image contains the `yast-travis-cpp` script which runs all the checks and tests.

The workflow is:

- Copy the sources into the `/usr/src/app` directory.
- If the code needs additional packages install them using the `zypper install`
  command from the local `Dockerfile`. If the package can be used by more modules
  you can add it into the base Docker image here.
- Run the `yast-ci-cpp` script. (Optionally you can use the `-x` and `-o`
  options to split the work into several smaller tasks and run them in parallel,
  see the [yast2-storage-ng example](
  https://github.com/yast/yast-storage-ng/blob/master/.travis.yml).)

## Examples

### GitHub Action Example

Save as `.github/workflows/ci.yml` file in your Git repository:

```yaml
# See https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions

name: CI

on: [push, pull_request]

jobs:
  Package:
    runs-on: ubuntu-latest
    container: registry.opensuse.org/yast/head/containers/yast-cpp:latest

    steps:

    - name: Git Checkout
      uses: actions/checkout@v2

    - name: Prepare System
      run: |
        zypper --non-interactive in --no-recommends \
          needed-package1 \
          needed-package2

    - name: Package Build
      run:  yast-ci-cpp
```

## Building the Image Locally

### Using the OSC Tool

From the Git sources:

```sh
# you need the rubygem-yast-rake package installed
rake osc:build
```

From the OBS checkout:

```sh
# check it out if not already present
osc co YaST:Head/ci-cpp-container
cd YaST:Head/ci-cpp-container

# build it
osc build containers
```

### Using the Docker tool

️:warning: This approach is not 100% the same as building the image with `osc` described above.
The `osc` build injects some special modifications to allow building the image inside
the OBS build environment.

:information_source:️ You should prefer using the `osc` method if possible, use the `docker`
build only as a fallback when the `osc` build is not possible or does not work in your environment.

```sh
docker build -t ci-cpp-container-test -f package/Dockerfile package/
```
