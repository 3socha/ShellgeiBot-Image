# Arm64 のみ
FROM ubuntu:24.04 AS apt-cache
RUN apt-get update

FROM ubuntu:24.04 AS builder
ENV DEBIAN_FRONTEND noninteractive
RUN rm -f /etc/apt/apt.conf.d/docker-clean; \
    echo 'Binary::apt::APT::Keep-Downloaded-Packages "true";' > /etc/apt/apt.conf.d/keep-cache
RUN echo 'APT::Install-Recommends "false";' > /etc/apt/apt.conf.d/no-install-recommends
RUN --mount=type=bind,target=/var/lib/apt/lists,from=apt-cache,source=/var/lib/apt/lists \
    --mount=type=cache,target=/var/cache/apt \
    apt-get install -y -qq build-essential ca-certificates curl git unzip libffi-dev libffi8ubuntu1 libgmp-dev libgmp10 libncurses-dev libnuma1

# Ubuntu 24.04 の ghc 9.4.7, cabal-install 3.8.1.0 では template-haskell-2.19.0.0 が入り、egison-4.1.3 => sweet-egison-0.1.1.3 => egison-pattern-src-th-mode => template-haskell>=2.2.0 && <2.17 とコンフリクトする
# RUN --mount=type=bind,target=/var/lib/apt/lists,from=apt-cache,source=/var/lib/apt/lists \
#     --mount=type=cache,target=/var/cache/apt \
#     apt-get install -y -qq ghc cabal-install

# ghc-8.10.7 => base-4.14.3.0; template-haskell-2.16.0.0 => base (>=4.11 && <4.15)
# GHCup; https://www.haskell.org/ghcup/install/#version-23
RUN curl -sfSL https://downloads.haskell.org/~ghcup/aarch64-linux-ghcup -o /usr/local/bin/ghcup
RUN chmod +x /usr/local/bin/ghcup
ENV GHCUP_CURL_OPTS -sfSL

# GHC
ENV GHC_VERSION 8.10.7
RUN ghcup install ghc $GHC_VERSION
RUN ghcup set ghc $GHC_VERSION

# Cabal
RUN ghcup install cabal
ENV PATH $PATH:/root/.ghcup/bin

# LLVM 9; https://gitlab.haskell.org/ghc/ghc/-/wikis/commentary/compiler/backends/llvm/installing#llvm-support

# Ubuntu 24.04 の apt リポジトリから; LLVM 14-18 のみ
# RUN --mount=type=bind,target=/var/lib/apt/lists,from=apt-cache,source=/var/lib/apt/lists \
#     --mount=type=cache,target=/var/cache/apt \
#     apt-get install -y -qq llvm-14

# LLVM の apt リポジトリから; 17-19 のみ
# RUN mkdir -p /usr/local/share/keyrings
# RUN curl -sfSL https://apt.llvm.org/llvm-snapshot.gpg.key | gpg --dearmor -o /usr/local/share/keyrings/llvm.gpg
# RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/local/share/keyrings/llvm.gpg] http://apt.llvm.org/noble/ llvm-toolchain-noble main" | tee /etc/apt/sources.list.d/llvm.list

# 事前ビルド済み LLVM 9; 依存する共有ライブラリのバージョンが合わない
# RUN curl -LO https://github.com/llvm/llvm-project/releases/download/llvmorg-9.0.1/clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz \
#     && tar xf clang+llvm-9.0.1-aarch64-linux-gnu.tar.xz \
#     && mv clang+llvm-9.0.1-aarch64-linux-gnu /usr/local/llvm
# ENV PATH $PATH:/usr/local/llvm/bin

# ソースから LLVM 9 をビルド; ビルドが通らない
# ENV LLVM_VERSION 9.0.1
# RUN --mount=type=bind,target=/var/lib/apt/lists,from=apt-cache,source=/var/lib/apt/lists \
#     --mount=type=cache,target=/var/cache/apt \
#     apt-get install -y -qq cmake python3
# WORKDIR /root
# RUN curl -sfSLO https://github.com/llvm/llvm-project/releases/download/llvmorg-$LLVM_VERSION/llvm-$LLVM_VERSION.src.tar.xz \
#     && tar xf llvm-$LLVM_VERSION.src.tar.xz \
#     && rm llvm-$LLVM_VERSION.src.tar.xz
# WORKDIR /root/builddir
# # RUN cmake /root/llvm-$LLVM_VERSION.src
# RUN --mount=type=cache,target=/root/builddir \
#     cmake /root/llvm-$LLVM_VERSION.src
# RUN --mount=type=cache,target=/root/builddir \
#     cmake --build . --parallel
# WORKDIR /
