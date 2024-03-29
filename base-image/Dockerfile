# syntax = docker/dockerfile:1.3

# Copyright 2020-2022 Datawire. All rights reserved.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

FROM golang:alpine3.15 as tel2-build

RUN apk add --no-cache gcc musl-dev

WORKDIR telepresence
COPY go.mod go.sum .
COPY cmd/ cmd/
COPY pkg/ pkg/
COPY rpc/ rpc/
COPY charts/ charts/
COPY build-output/version.txt .

RUN \
    --mount=type=cache,target=/root/.cache/go-build \
    --mount=type=cache,target=/go/pkg/mod \
    go build -o /usr/local/bin/ -trimpath -ldflags=-X=$(go list ./pkg/version).Version=$(cat version.txt) ./cmd/traffic/...

# The tel2 targer is the one that gets published. It aims to be a small as possible.
FROM alpine:3.15 as tel2

RUN apk add --no-cache ca-certificates iptables

# the traffic binary
COPY --from=tel2-build /usr/local/bin/traffic /usr/local/bin

RUN \
  mkdir /tel_app_mounts && \
  chgrp -R 0 /tel_app_mounts && \
  chmod -R g=u /tel_app_mounts && \
  mkdir -p /home/telepresence && \
  chgrp -R 0 /home/telepresence && \
  chmod -R g=u /home/telepresence && \
  chmod 0777 /home/telepresence

ENTRYPOINT ["traffic"]
CMD []
