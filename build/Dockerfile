FROM emscripten/emsdk:5.0.7 AS builder

ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get update && apt-get install -y --no-install-recommends \
        autoconf automake flex libfl-dev bison perl python3 \
        ca-certificates \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

ARG VERSION=5.048
ENV VERSION=${VERSION}
ARG SOURCE_SHA256=02d934b3f972c6d9b792350634d81eadfc9e61f347e3f3bdcaad40960b9fcb53

ENV ROOT=/src
ENV DIR=verilator-${VERSION}
ENV VROOT=/verilator
WORKDIR /src

ADD https://github.com/verilator/verilator/archive/refs/tags/v${VERSION}.tar.gz /src/verilator.tar.gz
RUN echo "${SOURCE_SHA256}  verilator.tar.gz" | sha256sum -c - \
 && tar -xzf verilator.tar.gz && rm verilator.tar.gz

WORKDIR /src/${DIR}

ENV CFLAGS="-O2"
ENV CXXFLAGS="-O2 -DVL_IGNORE_UNKNOWN_ARCH"

RUN autoconf

RUN emconfigure ./configure --host=wasm32-unknown-emscripten --prefix=${VROOT}

COPY js /src/${DIR}/js

RUN H="$(find /usr -name FlexLexer.h 2>/dev/null | head -1)" \
 && test -n "$H" \
 && cp "$H" /src/${DIR}/src/FlexLexer.h \
 && test -f /src/${DIR}/src/FlexLexer.h

RUN sed -i \
      's@#if defined(__linux) || defined(CPU_ZERO)  // Linux-like; assume we have pthreads etc@#if (defined(__linux) || defined(CPU_ZERO)) \&\& !defined(__EMSCRIPTEN__)  // Linux-like; assume we have pthreads etc@' \
      /src/${DIR}/include/verilatedos_c.h \
 && grep -q '!defined(__EMSCRIPTEN__)' /src/${DIR}/include/verilatedos_c.h

ENV VERILATOR_LDFLAGS="\
    -lnodefs.js -lworkerfs.js \
    --post-js /src/verilator-${VERSION}/js/post.js \
    --preload-file /src/verilator-${VERSION}/include@/verilator/include \
    -sWASM_BIGINT=1 \
    -sALLOW_MEMORY_GROWTH=1 \
    -sINITIAL_MEMORY=134217728 \
    -sSTACK_SIZE=8388608 \
    -sFORCE_FILESYSTEM=1 \
    -sEXIT_RUNTIME=0 \
    -sEXPORTED_RUNTIME_METHODS=callMain,FS,NODEFS,WORKERFS,ENV \
    -sINCOMING_MODULE_JS_API=noInitialRun,wasmBinary,locateFile,preRun,postRun,instantiateWasm,print,printErr,arguments,mainScriptUrlOrBlob \
    -sINVOKE_RUN=0 \
    -sMODULARIZE=1 \
    -sEXPORT_ES6=1 \
"

RUN emmake make -C src opt \
        EXEEXT=.mjs \
        LDFLAGS="${VERILATOR_LDFLAGS}" \
        -j"$(nproc)"

RUN MJS="$(find . -name 'verilator_bin.mjs' | head -1)" \
 && DIRP="$(dirname "$MJS")" \
 && test -f "$DIRP/verilator_bin.wasm" \
 && test -f "$DIRP/verilator_bin.data" \
 && mkdir -p /dist \
 && cp "$DIRP/verilator_bin.mjs"  /dist/verilator.mjs \
 && cp "$DIRP/verilator_bin.wasm" /dist/verilator.wasm \
 && cp "$DIRP/verilator_bin.data" /dist/verilator.data \
 && sed -i 's/verilator_bin\.wasm/verilator.wasm/g; s/verilator_bin\.data/verilator.data/g' /dist/verilator.mjs \
 && grep -q 'verilator\.wasm' /dist/verilator.mjs \
 && grep -q 'verilator\.data' /dist/verilator.mjs \
 && ! grep -q 'verilator_bin\.' /dist/verilator.mjs \
 && [ "$(head -c4 /dist/verilator.wasm | od -An -tx1 | tr -d ' \n')" = "0061736d" ] \
 && for f in verilator.mjs verilator.wasm verilator.data; do \
        [ "$(stat -c%s /dist/$f)" -gt 4096 ] || { echo "too small: $f" >&2; exit 1; }; \
    done \
 && ls -lh /dist

FROM scratch AS exportor
COPY --from=builder /dist /dist
