FROM golang:1.17 as build

RUN apt-get -y update && \
    apt-get -y install unzip

ARG PROTOC_VERSION=3.17.1
RUN wget -q -O protoc.zip "https://github.com/protocolbuffers/protobuf/releases/download/v${PROTOC_VERSION}/protoc-${PROTOC_VERSION}-linux-x86_64.zip" && \
    unzip -o protoc.zip bin/protoc -d /

ARG PROTOC_GEN_GO_VERSION=1.26.0
RUN go install "google.golang.org/protobuf/cmd/protoc-gen-go@v${PROTOC_GEN_GO_VERSION}"

ARG PROTOC_GEN_GO_GRPC_VERSION=1.1.0
RUN go install "google.golang.org/grpc/cmd/protoc-gen-go-grpc@v${PROTOC_GEN_GO_GRPC_VERSION}"

ARG GRPC_HEALTH_PROBE_VERSION=0.4.2
RUN wget -q -O /bin/grpc_health_probe "https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/v${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64" && \
    chmod +x /bin/grpc_health_probe

WORKDIR /src

COPY go.mod /src/
COPY go.sum /src/
RUN go mod download

COPY . /src/
RUN go generate ./...
RUN CGO_ENABLED=0 go build -o /bin/echo-grpc .

FROM alpine:3.13.2

ENV GRPC_GO_LOG_VERBOSITY_LEVEL=2
ENV GRPC_GO_LOG_SEVERITY_LEVEL=debug
ENV PORT=8080
EXPOSE 8080

COPY --from=build /bin/grpc_health_probe /bin/grpc_health_probe
COPY --from=build /bin/echo-grpc /bin/echo-grpc

ENTRYPOINT ["/bin/echo-grpc"]
