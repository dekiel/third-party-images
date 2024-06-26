# image builder base on golang:1.21.3-alpine3.18
FROM europe-docker.pkg.dev/kyma-project/prod/external/golang@sha256:926f7f7e1ab8509b4e91d5ec6d5916ebb45155b0c8920291ba9f361d65385806 as builder


ENV GO111MODULE=auto
ENV DISTRIBUTION_DIR /go/src/github.com/docker/distribution
ENV BUILDTAGS include_oss include_gcs

ARG REGISTRY_VERSION
ARG GOOS=linux
ARG GOARCH=amd64
ARG GOARM=6

RUN set -ex \
    && apk upgrade --no-cache \
    && apk add --no-cache make git file

WORKDIR $DISTRIBUTION_DIR
RUN git clone -b v${REGISTRY_VERSION} https://github.com/distribution/distribution $DISTRIBUTION_DIR

RUN CGO_ENABLED=0 make PREFIX=/go clean binaries && file ./bin/registry | grep "statically linked"

# result container base on alpine:3.17.4
FROM europe-docker.pkg.dev/kyma-project/prod/external/alpine@sha256:e95676db9e4a4f16f6cc01a8915368f82b018cc07aba951c1bd1db586c081388

RUN set -ex \
    && apk upgrade --no-cache \
    && apk add --no-cache ca-certificates apache2-utils

COPY --from=builder /go/src/github.com/docker/distribution/cmd/registry/config-dev.yml /etc/docker/registry/config.yml
COPY --from=builder /go/src/github.com/docker/distribution/bin/registry /bin/registry

VOLUME ["/var/lib/registry"]
EXPOSE 5000

LABEL source=git@github.com:kyma-project/kyma.git

ENTRYPOINT ["registry"]
CMD ["serve", "/etc/docker/registry/config.yml"]
