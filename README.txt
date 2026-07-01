verilator-wasm — Corresponding Source mirror
==================================================

This repository publishes the Corresponding Source for the WebAssembly
build of verilator (license: LGPL-3.0-only OR Artistic-2.0) used in edgetools.io.

Contents
  build/      our build recipe: Dockerfile + helper scripts/config/patches.
              Rebuild with:  docker build build/
  upstream/   the exact upstream source archive(s) the build fetched,
              byte-identical and sha256-verified (see below).

Upstream sources:
  verilator.tar.gz
    https://github.com/verilator/verilator/archive/refs/tags/v5.048.tar.gz
    sha256 02d934b3f972c6d9b792350634d81eadfc9e61f347e3f3bdcaad40960b9fcb53
