ARG BASE_IMAGE=centos:centos7
ARG SRC_DIR="/home/k8s-rbd/ceph-csi/"
ARG GO_ARCH
ARG CSI_IMAGE_NAME
ARG CSI_IMAGE_VERSION

FROM ${BASE_IMAGE} as builder
LABEL stage="build"

ARG RPM_DIR=/mnt/rpms/

RUN yum install -y dnf epel-release createrepo && \
    mkdir -p "${RPM_DIR}" && rm -f /etc/yum.repos.d/ceph.repo

COPY ./rpms "${RPM_DIR}"

RUN createrepo "${RPM_DIR}"

WORKDIR "${RPM_DIR}"

RUN echo $'[local] \n\
name            = Local install Ceph \n\
baseurl         = file:///mnt/rpms \n\
enabled         = 1 \n\
gpgcheck        = 0' >> /etc/yum.repos.d/local.repo

RUN yum install -y librbd-devel librados-devel libcephfs-devel ceph-common --enablerepo=cr

ARG GOLANG_VERSION=1.13.9
ARG GIT_COMMIT=3f0d2b3c4ffc2bf900363254ba7eda98705db910
ARG CSI_IMAGE_NAME
ARG CSI_IMAGE_VERSION
ARG GO_ARCH
ARG SRC_DIR
ARG GOROOT=/usr/local/go

RUN mkdir -p ${GOROOT} && \
    curl https://storage.googleapis.com/golang/go${GOLANG_VERSION}.linux-${GO_ARCH}.tar.gz | tar xzf - -C ${GOROOT} --strip-components=1

RUN dnf install /usr/bin/cc make -y

ENV GOROOT=${GOROOT} \
    GOPATH=/go \
    CGO_ENABLED=1 \
    GIT_COMMIT="${GIT_COMMIT}" \
    ENV_CSI_IMAGE_VERSION="${CSI_IMAGE_VERSION}" \
    ENV_CSI_IMAGE_NAME="${CSI_IMAGE_NAME}" \
    PATH="${GOROOT}/bin:${GOPATH}/bin:${PATH}"

WORKDIR ${SRC_DIR}

# Copy source directories
COPY . ${SRC_DIR}

# Build executable
RUN make cephcsi

FROM ${BASE_IMAGE}

ARG SRC_DIR

COPY --from=builder ${SRC_DIR}/_output/cephcsi /usr/local/bin/cephcsi
COPY --from=builder /usr/include /usr/include
COPY --from=builder /usr/lib64 /usr/lib64
COPY --from=builder /usr/lib /usr/lib
COPY --from=builder /usr/bin /usr/bin

LABEL maintainers="Tsung-Ju Lii" \
    version=${CSI_IMAGE_VERSION} \
    architecture=${GO_ARCH} \
    description="Ceph-CSI Plugin for QCS / Kubernetes"

ENTRYPOINT ["/usr/local/bin/cephcsi"]
