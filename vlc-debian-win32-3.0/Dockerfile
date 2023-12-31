FROM debian:stretch-20190326-slim

# If someone wants to use VideoLAN docker images on a local machine and does
# not want to be disturbed by the videolan user, we should not take an uid/gid
# in the user range of main distributions, which means:
# - Debian based: <1000
# - RPM based: <500 (CentOS, RedHat, etc.)
ARG VIDEOLAN_UID=499
# 8 cores ought to be enough for anybody
ARG CORES=8

ENV IMAGE_DATE=202110081601

ENV TARGET_TUPLE=i686-w64-mingw32
ENV TOOLCHAIN_PREFIX=/opt/gcc-$TARGET_TUPLE
ENV MINGW_PREFIX=$TOOLCHAIN_PREFIX/$TARGET_TUPLE
ENV PATH=$TOOLCHAIN_PREFIX/bin:/opt/cmake/bin:/opt/protobuf/bin:$PATH

COPY scripts/wait_process.sh /opt/wine/
COPY patches /patches

RUN set -x && \
    mkdir -p /usr/share/man/man1 && \
    addgroup --quiet --gid ${VIDEOLAN_UID} videolan && \
    adduser --quiet --uid ${VIDEOLAN_UID} --ingroup videolan videolan && \
    echo "videolan:videolan" | chpasswd && \
    echo "export TARGET_TUPLE=${TARGET_TUPLE}" >> /etc/profile.d/vlc_env.sh && \
    echo "export TOOLCHAIN_PREFIX=${TOOLCHAIN_PREFIX}" >> /etc/profile.d/vlc_env.sh && \
    echo "export MINGW_PREFIX=${MINGW_PREFIX}" >> /etc/profile.d/vlc_env.sh && \
    echo "export PATH=${TOOLCHAIN_PREFIX}/bin:\$PATH" >> /etc/profile.d/vlc_env.sh && \
    apt-get update -qq && apt-get install -qqy --no-install-suggests --no-install-recommends \
    git wget bzip2 file libwine-dev unzip libtool libtool-bin libltdl-dev pkg-config ant \
    build-essential automake texinfo ragel yasm p7zip-full autopoint \
    gettext dos2unix zip wine nsis curl gperf flex bison \
    libcurl4-gnutls-dev python3 python3-requests lftp gcc make procps ca-certificates \
    openjdk-8-jdk-headless \
    && \
    dpkg --add-architecture i386 && \
    apt-get update && apt-get -y install --no-install-suggests --no-install-recommends wine32 && \
    echo "deb http://deb.debian.org/debian buster main" > /etc/apt/sources.list.d/buster.list && \
    apt-get update && apt-get -y -t buster --no-install-suggests --no-install-recommends install nasm nsis meson && \
    rm -f /etc/apt/sources.list.d/buster.list && \
    apt-get clean -y && rm -rf /var/lib/apt/lists/* && \
    update-alternatives --install /usr/bin/python python /usr/bin/python3.5 1 && \
    \
    GCC_VERSION=6.4.0 \
    GCC_SHA256=850bf21eafdfe5cd5f6827148184c08c4a0852a37ccf36ce69855334d2c914d4 \
    BINUTILS_VERSION=2.27 \
    BINUTILS_SHA256=369737ce51587f92466041a97ab7d2358c6d9e1b6490b3940eb09fb0a9a6ac88 \
    MPFR_VERSION=3.1.6 \
    MPFR_SHA256=569ceb418aa935317a79e93b87eeb3f956cab1a97dfb2f3b5fd8ac2501011d62 \
    GMP_VERSION=6.1.1 \
    GMP_SHA256=d36e9c05df488ad630fff17edb50051d6432357f9ce04e34a09b3d818825e831 \
    MPC_VERSION=1.0.3 \
    MPC_SHA256=617decc6ea09889fb08ede330917a00b16809b8db88c29c31bfbb49cbf88ecc3 \
    && \
    mkdir /build/ && \
    cd /build && \
    mkdir $TOOLCHAIN_PREFIX && \
    mkdir $MINGW_PREFIX && \
    ln -s $MINGW_PREFIX $TOOLCHAIN_PREFIX/mingw && \
    wget -q http://ftp.gnu.org/gnu/binutils/binutils-$BINUTILS_VERSION.tar.bz2 && \
    echo $BINUTILS_SHA256 binutils-$BINUTILS_VERSION.tar.bz2 | sha256sum -c && \
    wget -q ftp://ftp.uvsq.fr/pub/gcc/releases/gcc-$GCC_VERSION/gcc-$GCC_VERSION.tar.xz && \
    echo $GCC_SHA256 gcc-$GCC_VERSION.tar.xz | sha256sum -c && \
    git config --global user.name "VideoLAN Buildbot" && \
    git config --global user.email buildbot@videolan.org && \
    git clone --depth=1 --branch v5.0.3 git://git.code.sf.net/p/mingw-w64/mingw-w64 && \
    tar xf gcc-$GCC_VERSION.tar.xz && \
    tar xf binutils-$BINUTILS_VERSION.tar.bz2 && \
    cd binutils-$BINUTILS_VERSION && mkdir build && cd build && \
    ../configure --prefix=$TOOLCHAIN_PREFIX --target=$TARGET_TUPLE \
                    --disable-werror --disable-multilib && make -j$CORES && make install-strip && \
    cd /build/mingw-w64/mingw-w64-headers && mkdir build && cd build && \
    ../configure --prefix=$MINGW_PREFIX \
                    --host=$TARGET_TUPLE --enable-secure-api && make install && \
    cd /build && \
    wget -q https://ftp.gnu.org/gnu/mpfr/mpfr-$MPFR_VERSION.tar.gz && \
    echo $MPFR_SHA256 mpfr-$MPFR_VERSION.tar.gz | sha256sum -c && \
    wget -q https://gmplib.org/download/gmp/gmp-$GMP_VERSION.tar.xz && \
    echo $GMP_SHA256 gmp-$GMP_VERSION.tar.xz | sha256sum -c && \
    wget -q ftp://ftp.gnu.org/gnu/mpc/mpc-$MPC_VERSION.tar.gz && \
    echo $MPC_SHA256 mpc-$MPC_VERSION.tar.gz | sha256sum -c && \
    tar xf mpfr-$MPFR_VERSION.tar.gz && \
    tar xf gmp-$GMP_VERSION.tar.xz && \
    tar xf mpc-$MPC_VERSION.tar.gz && \
    ln -s /build/mpfr-$MPFR_VERSION gcc-$GCC_VERSION/mpfr && \
    ln -s /build/gmp-$GMP_VERSION gcc-$GCC_VERSION/gmp && \
    ln -s /build/mpc-$MPC_VERSION gcc-$GCC_VERSION/mpc && \
    cd /build/gcc-$GCC_VERSION && patch -p0 < /patches/libstdcxx.patch && \
    patch -p0 < /patches/gcc-configure-ac.patch && \
    autoreconf -vif && cd libstdc++-v3/ && autoreconf && \
    cd /build/gcc-$GCC_VERSION && \
    mkdir build && cd build && \
    ../configure --prefix=$TOOLCHAIN_PREFIX \
                    --target=$TARGET_TUPLE \
                    --enable-languages=c,c++ \
                    --enable-lto \
                    --disable-shared \
                    --disable-multilib && \
                    make -j$CORES all-gcc && \
                    make install-gcc && \
    cd /build/mingw-w64/mingw-w64-crt && \
    mkdir build && cd build && \
    ../configure --prefix=$MINGW_PREFIX \
                    --host=$TARGET_TUPLE && \
    make -j$CORES && \
    make install && \
    cd /build/gcc-$GCC_VERSION/build && \
    make -j$CORES && \
    make install-strip && \
    cd /build/mingw-w64/mingw-w64-tools/widl && \
    mkdir build && cd build && \
    ../configure --prefix=$TOOLCHAIN_PREFIX --target=$TARGET_TUPLE && \
    make -j$CORES && \
    make install && \
    cd /build/ && git clone --recursive https://code.videolan.org/videolan/breakpad.git && \
    cd breakpad && autoreconf -vif && mkdir build && cd build && \
    ../configure --enable-tools --disable-processor --prefix=/opt/breakpad && \
    make -j$CORES && make install && \
    \
    chmod +x /opt/wine/wait_process.sh && \
    wget -q https://raw.githubusercontent.com/Winetricks/winetricks/d30478da6981e0842956e5b326339b52540efaec/src/winetricks -O /opt/wine/winetricks && \
    WINETRICKS_SHA256=01ebb56d22f54467343700eedfe283d82a47e67bbb6c2d68b7da743f41bb78a3 && \
    echo $WINETRICKS_SHA256 /opt/wine/winetricks | sha256sum -c && \
    chmod +x /opt/wine/winetricks && \
    WINE_MONO_VERSION=4.5.6 && mkdir -p /usr/share/wine/mono && \
    wget -q "https://download.videolan.org/contrib/wine-mono/wine-mono-$WINE_MONO_VERSION.msi" -O /usr/share/wine/mono/wine-mono-$WINE_MONO_VERSION.msi && \
    WINE_MONO_SHA256=ac681f737f83742d786706529eb85f4bc8d6bdddd8dcdfa9e2e336b71973bc25 && \
    echo $WINE_MONO_SHA256 /usr/share/wine/mono/wine-mono-$WINE_MONO_VERSION.msi | sha256sum -c && \
    chmod +x /usr/share/wine/mono/wine-mono-$WINE_MONO_VERSION.msi && \
    cd /build && \
    CMAKE_VERSION=3.17.0 && \
    CMAKE_SHA256=b74c05b55115eacc4fa2b77a814981dbda05cdc95a53e279fe16b7b272f00847 && \
    wget -q http://www.cmake.org/files/v3.17/cmake-$CMAKE_VERSION.tar.gz && \
    echo $CMAKE_SHA256 cmake-$CMAKE_VERSION.tar.gz | sha256sum -c && \
    tar xzf cmake-$CMAKE_VERSION.tar.gz && \
    cd cmake-$CMAKE_VERSION && ./configure --prefix=/opt/cmake/ --parallel=$CORES --no-qt-gui -- \
        -DCMAKE_USE_OPENSSL:BOOL=OFF -DBUILD_TESTING:BOOL=OFF && make -j$CORES && make install && \
    cd /build && \
    PROTOBUF_VERSION=3.1.0 && \
    PROTOBUF_SHA256=51ceea9957c875bdedeb1f64396b5b0f3864fe830eed6a2d9c066448373ea2d6 && \
    wget -q https://github.com/google/protobuf/releases/download/v$PROTOBUF_VERSION/protobuf-cpp-$PROTOBUF_VERSION.tar.gz && \
    echo $PROTOBUF_SHA256 protobuf-cpp-$PROTOBUF_VERSION.tar.gz | sha256sum -c && \
    tar xzfo protobuf-cpp-$PROTOBUF_VERSION.tar.gz && \
    cd protobuf-$PROTOBUF_VERSION && \
    ./configure --prefix=/opt/protobuf --disable-shared --enable-static && make -j$CORES && make install && \
    cd / && rm -rf /build

USER videolan
RUN wine wineboot --init && \
    /opt/wine/wait_process.sh wineserver && \
    /opt/wine/winetricks --unattended dotnet40 dotnet_verifier && \
    /opt/wine/wait_process.sh wineserver && \
    wget -q https://download.videolan.org/contrib/wix/wix-3.5.msi -O ~/wix.msi && \
    WIX_SHA256=621b70e8761d5b940d8c32a42b0e92fd55767f8908a9b32e06bb3d12a30bc47b && \
    echo $WIX_SHA256 ~/wix.msi | sha256sum -c && \
    wine msiexec /i ~/wix.msi && \
    cd ~/ && \
    rm -f ~/wix.msi && \
    rm -rf ~/.cache/winetricks
