FROM golang:1-alpine AS build

ARG VERSION="1.6.5"
ARG CHECKSUM="df2e61a857eb7735258fd230e096e254596f18a89b5504dc2ac687798403343a"

ADD https://github.com/coredns/coredns/archive/v$VERSION.tar.gz /tmp/coredns.tar.gz

RUN [ "$CHECKSUM" = "$(sha256sum /tmp/coredns.tar.gz | awk '{print $1}')" ] && \
  tar -C /tmp -xf /tmp/coredns.tar.gz && \
  mv /tmp/coredns-$VERSION /tmp/coredns

COPY plugin.cfg /tmp/coredns/

RUN apk add make bash git ca-certificates 	unbound-dev unbound gcc libc-dev && \
  cd /tmp/coredns && \
  go generate && \
  go build -v -ldflags="-s -w -X github.com/coredns/coredns/coremain.GitCommit=" -o coredns && \
  echo "nogroup:*:100:nobody" > /tmp/group && \
  echo "nobody:*:100:100:::" > /tmp/passwd


FROM scratch
COPY --from=build /lib /lib
COPY --from=build /usr/lib/libunbound* /usr/lib/libevent* /lib/
COPY --from=build --chown=100:100 /tmp/coredns/coredns /
COPY --from=build --chown=100:100 /etc/ssl/certs/ca-certificates.crt /etc/ssl/certs/
COPY --from=build --chown=100:100 /tmp/group \
  /tmp/passwd \
  /etc/

USER 100:100
ENTRYPOINT ["/coredns"]