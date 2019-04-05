# The YaST C++ Testing Image

[![Build Status](https://travis-ci.org/yast/ci-cpp-container.svg?branch=master)](https://travis-ci.org/yast/ci-cpp-container)

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
- Run the `yast-travis-cpp` script. (Optionally you can use the `-x` and `-o`
  options to split the work into several smaller tasks and run them in parallel,
  see the [yast2-storage-ng example](
  https://github.com/yast/yast-storage-ng/blob/master/.travis.yml).)

## Examples

### `Dockerfile` example

```Dockerfile
FROM registry.opensuse.org/yast/head/containers/yast-cpp

# optionally install additional packages if needed:
# RUN zypper --non-interactive install --no-recommends \
#  libxml2-devel \
#  yast2-core-devel

# copy the sources into the image
COPY . /usr/src/app
```

### `.travis.yml` Example

```yaml
sudo: required
language: bash
services:
  - docker

before_install:
  - docker build -t yast-foo-image .
  # list the installed packages (just for easier debugging)
  - docker run --rm -it yast-foo-image rpm -qa | sort

script:
  # the "yast-travis-cpp" script is included in the base yast-ruby image
  # see https://github.com/yast/ci-cpp-container/blob/master/package/yast-travis-cpp
  - docker run -it --rm -e TRAVIS=1 -e TRAVIS_JOB_ID="$TRAVIS_JOB_ID" yast-foo-image yast-travis-cpp
```

(Replace `foo` with your package name to avoid collisions with the other packages
when running the same commands locally.)


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
