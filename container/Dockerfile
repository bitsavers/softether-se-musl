FROM alpine:3.20 AS builder

ARG SOFTETHER_GIT_TAG="v4.43-9799-beta"

ENV LANG="en_US.UTF-8" \
    DEBUG="no" \
    CC="gcc" \
    CFLAGS="-Wno-error=incompatible-pointer-types"

# https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/blob/master/src/BUILD_UNIX.TXT
# https://github.com/SoftEtherVPN/SoftEtherVPN_Stable/blob/master/configure#L117
RUN set -ex; \
    apk add --no-cache --update \
        git coreutils binutils gcc musl-dev make file patch fortify-headers ncurses-dev openssl-dev readline-dev zlib-dev gnu-libiconv-dev; \
    mkdir -p /app/softether-vpn; \
    git clone --depth 1 --branch ${SOFTETHER_GIT_TAG} https://github.com/SoftEtherVPN/SoftEtherVPN_Stable.git /app/softether-vpn; \
    cd /app/softether-vpn; \
    git submodule init && git submodule update; \
    apkArch="$(apk --print-arch)"; \
    case "$apkArch" in \
        armhf) \
            CPU_BITS="32bit_nobits"; \
            echo "armv6" ;; \
        armv7) \
            CPU_BITS="32bit_nobits"; \
            echo "armv7" ;; \
        aarch64) \
            CPU_BITS="64bit_nobits"; \
            echo "arm64" ;; \
        x86) \
            CPU_BITS="32bit"; \
            echo "x86" ;; \
        x86_64) \
            CPU_BITS="64bit"; \
            echo "x86_64" ;; \
        *) echo >&2 "error: unsupported architecture: $apkArch"; exit 1 ;; \
    esac; \
    cd /app/softether-vpn; \
    cp /app/softether-vpn/src/makefiles/linux_${CPU_BITS}.mak /app/softether-vpn/Makefile; \
    make -j$(nproc) ; \
    make install; \
    make clean;


FROM scratch AS dist
# COPY --from=builder /usr/bin/ /usr/bin/
COPY --from=builder /usr/vpnserver/ /usr/vpnserver/
COPY --from=builder /usr/vpnbridge/ /usr/vpnbridge/
COPY --from=builder /usr/vpnclient/ /usr/vpnclient/
COPY --from=builder /usr/vpncmd/ /usr/vpncmd/
