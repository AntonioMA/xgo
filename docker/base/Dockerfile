# Go cross compiler (xgo): Base cross-compilation layer
# Copyright (c) 2014 Péter Szilágyi. All rights reserved.
#
# Released under the MIT license.

FROM ubuntu:16.04

# Configure the Go environment, since it's not going to change
ENV PATH   /usr/local/go/bin:$PATH
ENV GOPATH /go


# Inject the remote file fetcher and checksum verifier
ADD fetch.sh /fetch.sh
ENV FETCH /fetch.sh
RUN chmod +x $FETCH

# Make sure apt-get is up to date and dependent packages are installed
RUN \
  apt-get update && \
  apt-get install -y automake autogen build-essential ca-certificates                    \
    gcc-5-multilib g++-5-multilib gcc-mingw-w64 g++-mingw-w64 clang llvm-dev             \
    libtool libxml2-dev uuid-dev libssl-dev pkg-config patch                             \
    make xz-utils cpio wget zip unzip p7zip git                                          \
    --no-install-recommends

# Fix any stock package issues
RUN ln -s /usr/include/asm-generic /usr/include/asm

# Configure the container for OSX cross compilation
ENV OSX_SDK     MacOSX10.11.sdk
ENV OSX_NDK_X86 /usr/local/osx-ndk-x86

# Inject the new Go root distribution downloader and bootstrapper
ADD bootstrap_pure.sh /bootstrap_pure.sh
ENV BOOTSTRAP_PURE /bootstrap_pure.sh
RUN chmod +x $BOOTSTRAP_PURE

# Inject the C dependency cross compiler
ADD build_deps.sh /build_deps.sh
ENV BUILD_DEPS /build_deps.sh
RUN chmod +x $BUILD_DEPS

# Inject the container entry point, the build script
ADD build.sh /build.sh
ENV BUILD /build.sh
RUN chmod +x $BUILD

RUN git clone https://github.com/tpoechtrager/osxcross.git

ENV OSX_SDK_PATH MacOSX10.11.sdk.tar.xz
COPY ${OSX_SDK_PATH} /osxcross/tarballs

RUN  sed -i -e 's|-march=native||g' /osxcross/build_clang.sh /osxcross/wrapper/build.sh && \
  UNATTENDED=yes OSX_VERSION_MIN=10.10 /osxcross/build.sh                            && \
  mv /osxcross/target $OSX_NDK_X86                                                   && \
  \
  rm -rf /osxcross

ENV PATH $OSX_NDK_X86/bin:$PATH

# Configure the root Go distribution and bootstrap based on it
ENV GO_VERSION 1210

RUN \
  export ROOT_DIST=https://storage.googleapis.com/golang/go1.12.1.linux-amd64.tar.gz  && \
  export ROOT_DIST_SHA=2a3fdabf665496a0db5f41ec6af7a9b15a49fbe71a85a50ca38b1f13a103aeec && \
  \
  $BOOTSTRAP_PURE
