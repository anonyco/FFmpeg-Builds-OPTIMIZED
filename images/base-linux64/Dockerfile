ARG GH_REPO=ghcr.io/btbn/ffmpeg-builds

FROM scratch AS downloader
ARG GH_REPO

ADD https://github.com/crosstool-ng/crosstool-ng.git /downloads/crosstool-ng

ADD https://github.com/yugr/Implib.so.git /downloads/Implib.so


FROM $GH_REPO/base:latest
ARG GH_REPO

ENV FFBUILD_TOOLCHAIN=x86_64-ffbuild-linux-gnu\
    FFBUILD_RUST_TARGET="x86_64-unknown-linux-gnu"

RUN --mount=type=bind,src=.,dst=/input\
 --mount=type=bind,ro,from=downloader,source=/downloads,target=/downloads\
 cp -vdpR /downloads/crosstool-ng /ct-ng &&\
 cp -v /input/toolchain.cmake / &&\
 cp -v /input/cross.meson / &&\
 cd /ct-ng &&\
 ./bootstrap &&\
 ./configure --enable-local &&\
 make -j$(nproc) &&\
 cat /input/.config >.config &&\
 ./ct-ng build &&\
 cd / &&\
 cp -vdpR /downloads/Implib.so /opt/implib &&\
 /input/gen-implib.sh /usr/bin/gen-implib &&\
  find /opt/ct-ng -type l \
    -and -name '*.so' \
    -and -not -ipath '*plugin*' \
    -and -not -name 'libdl.*' \
    -and -not -name 'libc.*' \
    -and -not -name 'libm.*' \
    -and -not -name 'libmvec.*' \
    -and -not -name 'librt.*' \
    -and -not -name 'libpthread.*' \
    -delete && \
  find /opt/ct-ng \
    -name 'libdl.a' \
    -or -name 'libc.a' \
    -or -name 'libm.a' \
    -or -name 'libmvec.a' \
    -or -name 'librt.a' \
    -or -name 'libpthread.a' \
    -delete && \
  mkdir /opt/ffbuild &&\
  rustup default nightly && \
  echo "[unstable]\ntarget-applies-to-host = true\nhost-config = true\n" > "$CARGO_HOME"/config.toml && \
  echo "[target.$FFBUILD_RUST_TARGET]\nlinker = \"${FFBUILD_TOOLCHAIN}-gcc\"\nar = \"${FFBUILD_TOOLCHAIN}-gcc-ar\"\n" >> "$CARGO_HOME"/config.toml && \
  echo "[target.host]\nlinker = \"gcc\"\nar = \"ar\"\n" >> "$CARGO_HOME"/config.toml

ENV\
  PATH="/opt/ct-ng/bin:${PATH:-/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin}"\
  FFBUILD_TARGET_FLAGS="--pkg-config=pkg-config --cross-prefix=${FFBUILD_TOOLCHAIN}- --arch=x86_64 --target-os=linux"\
  FFBUILD_CROSS_PREFIX="${FFBUILD_TOOLCHAIN}-"\
  FFBUILD_PREFIX=/opt/ffbuild\
  FFBUILD_CMAKE_TOOLCHAIN=/toolchain.cmake\
  PKG_CONFIG=pkg-config\
  PKG_CONFIG_LIBDIR=/opt/ffbuild/lib/pkgconfig:/opt/ffbuild/share/pkgconfig\
  CC="${FFBUILD_TOOLCHAIN}-gcc"\
  CXX="${FFBUILD_TOOLCHAIN}-g++"\
  LD="${FFBUILD_TOOLCHAIN}-ld"\
  AR="${FFBUILD_TOOLCHAIN}-gcc-ar"\
  RANLIB="${FFBUILD_TOOLCHAIN}-gcc-ranlib"\
  NM="${FFBUILD_TOOLCHAIN}-gcc-nm"\
  CFLAGS="-I/opt/ffbuild/include -O3 -fno-unroll-loops -fno-align-functions -fno-align-jumps -fno-align-labels -fno-align-loops -fno-ipa-cp-clone -fipa-pta -pipe -fPIC -DPIC -mtune=native -frename-registers -ftree-vectorize -fwrapv -fivopts -fomit-frame-pointer -g0 -DNDEBUG -U_FORTIFY_SOURCE -fno-asynchronous-unwind-tables -fno-stack-protector -fno-stack-clash-protection -fcf-protection=none -mharden-sls=none -fvisibility-inlines-hidden -w -pthread"\
  CXXFLAGS="-I/opt/ffbuild/include -O3 -fno-unroll-loops -fno-align-functions -fno-align-jumps -fno-align-labels -fno-align-loops -fno-ipa-cp-clone -fipa-pta -pipe -fPIC -DPIC -mtune=native -frename-registers -ftree-vectorize -fwrapv -fivopts -fomit-frame-pointer -g0 -DNDEBUG -U_FORTIFY_SOURCE -fno-asynchronous-unwind-tables -fno-stack-protector -fno-stack-clash-protection -fcf-protection=none -mharden-sls=none -fvisibility-inlines-hidden -w -pthread"\
  LDFLAGS="-L/opt/ffbuild/lib -O3 -pipe -Wl,-O1,--relax,--as-needed,-z,relro,-z,lazy -pthread -lm"\
  STAGE_CFLAGS="-fvisibility=hidden -fno-semantic-interposition"\
  STAGE_CXXFLAGS="-fvisibility=hidden -fno-semantic-interposition"

