FROM ligato/vpp-base:24.02@sha256:2cd129981e19a8a5a2eeb55c60f6c23649d1f2ee7160a60f6357f8bb28beac49 as builder
SHELL ["/bin/bash", "-o", "pipefail", "-c"]
COPY . /root/userspace-cni-network-plugin
WORKDIR /root/userspace-cni-network-plugin
RUN apt-get update -y \
    && DEBIAN_FRONTEND=noninteractive apt-get install --no-install-recommends -y binutils bash wget make git \
    && wget -qO- https://golang.org/dl/go1.20.1.linux-amd64.tar.gz | tar -C /usr/local -xz \
    && rm -rf /var/lib/apt/lists/*
ENV PATH="${PATH}:/usr/local/go/bin"
RUN go mod download \
    && go get go.fd.io/govpp/binapigen/vppapi@v0.7.0 \
    && make generate \
    && go mod tidy \
    && make generate-bin
#End of builder container
# Copy build userspace cni bin to a small deployer container
FROM alpine:3.20@sha256:77726ef6b57ddf65bb551896826ec38bc3e53f75cdde31354fbffb4f25238ebd
RUN mkdir -p /root/userspace-cni-network-plugin/userspace
COPY --from=builder /root/userspace-cni-network-plugin/userspace/userspace /root/userspace-cni-network-plugin/userspace/userspace
CMD ["cp", "-rf", "/root/userspace-cni-network-plugin/userspace/userspace", "/opt/cni/bin"]
