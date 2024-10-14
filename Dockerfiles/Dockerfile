# Build arguments
ARG SOURCE_CODE=.


FROM registry.redhat.io/ubi8/go-toolset:1.21 as builder

USER root
WORKDIR /workspace
ARG SOURCE_CODE


## Copy the Go Modules manifests
RUN cp ${SOURCE_CODE}/go.mod go.mod && \
    cp ${SOURCE_CODE}/go.sum go.sum

## Copy the manifests
RUN cp -r ${SOURCE_CODE}/prefetched-manifests/* /opt/manifests/

## Copy the go source
RUN cp ${SOURCE_CODE}/main.go main.go && \
    cp -r ${SOURCE_CODE}/apis/ apis/ && \
    cp -r ${SOURCE_CODE}/controllers/ controllers/ && \
    cp -r ${SOURCE_CODE}/pkg/ pkg/ && \
    cp -r ${SOURCE_CODE}/components/ components/

## Copy manifests
RUN mkdir -p /opt/manifests/monitoring/ /opt/manifests/partners/ /opt/manifests/osd-configs && \
    cp -r ${SOURCE_CODE}/config/monitoring/ /opt/manifests/ && \
    cp -r ${SOURCE_CODE}/config/partners/ /opt/manifests/ && \
    cp -r ${SOURCE_CODE}/config/osd-configs/ /opt/manifests/

# Build
ENV CGO_ENABLED=1
RUN GOOS=linux GOARCH=amd64 go build -a -o manager main.go

FROM registry.redhat.io/ubi8/ubi-minimal:latest

## Build args to be used at this step

## Add package for useradd util
RUN microdnf install -y shadow-utils &&\
    microdnf clean all

## Create a non-root user with UID 1001
RUN useradd --uid 1001 --create-home --user-group --system rhods

WORKDIR /

## Copy manager binary
COPY --from=builder /workspace/manager .

## Copy manifests
COPY --from=builder /opt/manifests /opt/manifests
RUN chown -R 1001:0 /opt/manifests &&\
    chmod -R g=u /opt/manifests

## Switch to a non-root user, but need to use numeric because of runAsNonRoot is added in CSV
USER 1001

ENTRYPOINT [ "/manager" ]

LABEL name="managed-open-data-hub/odh-rhel8-operator" \
      summary="odh-operator" \
      io.openshift.expose-services="" \
      io.k8s.display-name="odh-operator" \
      maintainer="['managed-open-data-hub@redhat.com']" \
      description="odh-operator" \
      com.redhat.license_terms="https://www.redhat.com/licenses/Red_Hat_Standard_EULA_20191108.pdf"
