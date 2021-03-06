# Copyright 2018 The Outline Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# Newer node images have no valid content trust data.
# Pin the image node:12.16.3-alpine (linux/amd64) by hash.
# See versions at https://hub.docker.com/_/node?tab=tags&name=alpine
FROM node@sha256:12b2154fb459fa5f42c54771524609db041e7ef3465935d0ca82940d2d72669d

# Versions can be found at https://github.com/Jigsaw-Code/outline-ss-server/releases
ARG SS_VERSION=1.1.0

# Save metadata on the software versions we are using.
LABEL shadowbox.node_version=12.16.3
LABEL shadowbox.outline-ss-server_version="${SS_VERSION}"

ARG GITHUB_RELEASE
LABEL shadowbox.github.release="${GITHUB_RELEASE}"

# We use curl to detect the server's public IP. We need to use the --date option in `date` to
# safely grab the ip-to-country database
RUN apk add --no-cache --upgrade coreutils curl

COPY src/shadowbox/scripts scripts/
COPY src/shadowbox/scripts/update_mmdb.sh /etc/periodic/weekly/update_mmdb

RUN /etc/periodic/weekly/update_mmdb

WORKDIR /root/shadowbox

RUN mkdir bin && curl -SsL https://github.com/Jigsaw-Code/outline-ss-server/releases/download/v${SS_VERSION}/outline-ss-server_${SS_VERSION}_linux_x86_64.tar.gz | tar xz -C bin outline-ss-server
COPY third_party/prometheus/prometheus ./bin/
COPY src/shadowbox/package.json .
COPY yarn.lock .

RUN yarn install --prod

# Install management service
COPY build/shadowbox/app app/

# Create default state directory.
RUN mkdir -p /root/shadowbox/persisted-state

COPY src/shadowbox/docker/cmd.sh /

CMD /cmd.sh
