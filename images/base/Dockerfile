FROM scratch AS downloader

#ADD /downloads/setup_lts.x https://deb.nodesource.com/setup_lts.x
ADD /downloads/node-linux-x64.xz https://nodejs.org/dist/latest/node-v23.8.0-linux-x64.tar.xz

ADD /downloads/sh.rustup.rs https://sh.rustup.rs

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive\
 CARGO_HOME="/opt/cargo"\
 RUSTUP_HOME="/opt/rustup"\
 PATH="/opt/cargo/bin:${PATH}"\
 HOST_CC="gcc" \
 HOST_CXX="g++" \
 HOST_CFLAGS="-O3 -fno-unroll-loops -fno-align-functions -fno-align-jumps -fno-align-labels -fno-align-loops -fno-ipa-cp-clone -fipa-pta -pipe -fwrapv -fivopts -fomit-frame-pointer -g0 -DNDEBUG -U_FORTIFY_SOURCE -fno-asynchronous-unwind-tables -w" \
 HOST_CXXFLAGS="-O3 -fno-unroll-loops -fno-align-functions -fno-align-jumps -fno-align-labels -fno-align-loops -fno-ipa-cp-clone -fipa-pta -pipe -fwrapv -fivopts -fomit-frame-pointer -g0 -DNDEBUG -U_FORTIFY_SOURCE -fno-asynchronous-unwind-tables -w"

RUN --mount=src=.,dst=/input\
 --mount=type=bind,ro,from=downloader,source=/downloads,target=/downloads\
 printf '%s\n' 'Types: deb' 'URIs: http://archive.ubuntu.com/ubuntu/' 'Suites: noble noble-updates noble-backports' 'Components: main universe' 'Signed-By: /usr/share/keyrings/ubuntu-archive-keyring.gpg' '' >/etc/apt/sources.list.d/ubuntu.sources &&\
 bash /downloads/setup_lts.x &&\
 apt-get -y upgrade build-essential ca-certificates yasm nasm\
  xxd pkgconf curl wget unzip zip git subversion mercurial rsync jq\
  autoconf automake libtool libtool-bin autopoint gettext cmake clang meson ninja-build\
  texinfo texi2html help2man flex bison groff\
  gperf itstool ragel libc6-dev zlib1g-dev libssl-dev\
  gtk-doc-tools gobject-introspection gawk\
  ocaml ocamlbuild libnum-ocaml-dev indent p7zip-full\
  python3-setuptools python3-jinja2 python3-jsonschema python3-apt python-is-python3 &&\
 update-ca-certificates &&\
 tar -C /usr --strip-components 1 -xvJf /downloads/node-linux-x64.xz &&\
 git config --global user.email "builder@localhost" &&\
 git config --global user.name "Builder" &&\
 git config --global advice.detachedHead false &&\
 bash /downloads/sh.rustup.rs -y --no-modify-path && \
 cargo install cargo-c && rm -rf "${CARGO_HOME}"/registry "${CARGO_HOME}"/git &&\
 cd /input &&\
 for s in *.sh; do cp -p ./$s /usr/bin/${s%.sh}; done &&\
 cd /usr/bin &&\
 ln -sfT gcc cc &&\
 ln -sfT g++ c++

