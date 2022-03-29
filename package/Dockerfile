
#!BuildTag: yast-cpp

# This actually is registry.opensuse.org/yast/head/images/opensuse/tumbleweed:latest
# when building in OBS YaST:Head!
FROM opensuse/tumbleweed

# do not install the files marked as documentation (use "rpm --excludedocs")
RUN sed -i -e "s/^.*rpm.install.excludedocs.*/rpm.install.excludedocs = yes/" /etc/zypp/zypp.conf

# import the YaST OBS GPG key
COPY YaST:Head.pub /usr/share/gpg-keys/
RUN rpm --import /usr/share/gpg-keys/YaST:Head.pub

# Prefer the packages from the YaST:Head repository
# TODO -p 50 does not work, but should be usless anyway, just like --refresh
RUN zypper addrepo --refresh http://download.opensuse.org/repositories/YaST:/Head/openSUSE_Tumbleweed yast

RUN zypper --non-interactive install --no-recommends \
  aspell-en \
  autoconf \
  automake \
  bison \
  boost-devel \
  brp-check-suse \
  build \
  cmake \
  dejagnu \
  docbook-xsl-stylesheets \
  doxygen \
  fdupes \
  flex \
  gcc-c++ \
  git \
  grep \
  hwinfo-devel \
  libtool \
  libxslt \
  obs-service-source_validator \
  openSUSE-release-ftp \
  rpm-build \
  ruby-devel \
  "rubygem(ruby:3.1.0:fast_gettext)" \
  "rubygem(ruby:3.1.0:gettext)" \
  "rubygem(ruby:3.1.0:raspell)" \
  "rubygem(ruby:3.1.0:rspec)" \
  "rubygem(ruby:3.1.0:rubocop)" \
  "rubygem(ruby:3.1.0:simplecov)" \
  "rubygem(ruby:3.1.0:yard)" \
  "rubygem(ruby:3.1.0:yast-rake)" \
  screen \
  sgml-skel \
  ShellCheck \
  update-desktop-files \
  which \
  yast2-core-devel \
  yast2-devtools \
  yast2-ycp-ui-bindings-devel \
  && zypper clean --all \
  && rm -rf /usr/lib*/ruby/gems/*/cache/ \
  && rm -rf /usr/share/doc/ \
  && rpm -e --nodeps kbd kbd-legacy \
  && find /usr/lib/locale/* -maxdepth 1 | grep -v -E "(en_US|cs_CZ|es_ES|de_DE|C.utf8)" | xargs rm -rf \
  && find /usr/share/locale -name "*.mo" -delete


COPY yast-ci-cpp /usr/local/bin/
RUN chmod a+x /usr/local/bin/yast-ci-cpp

# just a backward compatible symlink, should be removed in the future...
RUN ln -s yast-ci-cpp /usr/local/bin/yast-travis-cpp

ENV LC_ALL=en_US.UTF-8
RUN mkdir -p /usr/src/app
WORKDIR /usr/src/app

# run some smoke tests to make sure there is no serious issue with the image
RUN /usr/lib/YaST2/bin/y2base --help && c++ --version && rake -r yast/rake -V
