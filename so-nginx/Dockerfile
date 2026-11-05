# Copyright Security Onion Solutions, LLC

#    This program is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
#
#    This program is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
#
#    You should have received a copy of the GNU General Public License
#    along with this program.  If not, see <http://www.gnu.org/licenses/>.

# Navigator build stage
FROM docker.io/library/node:26.12.1-alpine@sha256:40cd011b55415bc894e45b02fdd5a5dfde056921b7d23a64c74f80497af48797 AS navigator-builder

# renovate: datasource=github-tags depName=attack-navigator packageName=mitre-attack/attack-navigator
ARG NAVIGATOR_VERSION=v5.3.2
ARG NAVIGATOR_COMMIT=76bf61f12f68b80a1eaeb3a2f9d3a816efc22ce1
# renovate: datasource=github-tags depName=attack-enterprise-data packageName=mitre-attack/attack-stix-data extractVersion=^v(?<version>.+)$
ARG ATTACK_VERSION=16.1
# sha256-of: https://raw.githubusercontent.com/mitre-attack/attack-stix-data/v${ATTACK_VERSION}/enterprise-attack/enterprise-attack-${ATTACK_VERSION}.json
ARG ATTACK_SHA256=8423d8dac3fc2feb825bb07d26e5f5d905e08a88f6fe4652cc20834cbe982813

RUN apk add git wget

RUN git clone https://github.com/mitre-attack/attack-navigator.git && \
    git -C attack-navigator -c advice.detachedHead=false checkout ${NAVIGATOR_COMMIT}

RUN wget -O /tmp/enterprise-attack.json https://raw.githubusercontent.com/mitre-attack/attack-stix-data/v${ATTACK_VERSION}/enterprise-attack/enterprise-attack-${ATTACK_VERSION}.json && \
    echo "${ATTACK_SHA256}  /tmp/enterprise-attack.json" | sha256sum -c -

WORKDIR /attack-navigator/nav-app/

RUN npm ci && npx --no-install ng build

# Delete base href line from index html to fix url path issues
RUN sed -i '/<base href="\/"\/>/d' ./dist/browser/index.html

###################################

FROM docker.io/library/nginx:1.31.8-alpine-slim@sha256:f45171d36c76a8b2f5ef97108a3a8a2294248a6578a0a180edc2d78f7d2edc26
HEALTHCHECK --interval=5m --timeout=3s CMD curl --fail http://localhost/ || exit 1
LABEL maintainer "Security Onion Solutions, LLC"
LABEL description "Security Onion Core Functions Docker"

RUN apk add --no-cache curl

# renovate: datasource=github-releases depName=cyberchef packageName=gchq/CyberChef extractVersion=^v(?<version>.+)$
ARG CYBERCHEF_VERSION=11.5.0
# release-asset-of: gchq/CyberChef v${CYBERCHEF_VERSION} CyberChef_(?<CYBERCHEF_HASH>[0-9a-f]+)\.zip
ARG CYBERCHEF_HASH=8cd426dd4f40f1423912d5fad91b578a86a65112
# sha256-of: https://github.com/gchq/CyberChef/releases/download/v${CYBERCHEF_VERSION}/CyberChef_${CYBERCHEF_HASH}.zip
ARG CYBERCHEF_SHA256=f6478925d3eaa16ec08626a85f5b85eed10d531a5e18700615fed7ecb024cccf

RUN mkdir -p /opt/socore/html/navigator
COPY --from=navigator-builder /attack-navigator/nav-app/dist/browser /opt/socore/html/navigator
COPY --from=navigator-builder /tmp/enterprise-attack.json /opt/socore/html/navigator/assets/mitre/enterprise-attack.json

RUN mkdir -p /opt/socore/html/cyberchef && \
    wget -O /tmp/CyberChef.zip https://github.com/gchq/CyberChef/releases/download/v${CYBERCHEF_VERSION}/CyberChef_${CYBERCHEF_HASH}.zip && \
    echo "${CYBERCHEF_SHA256}  /tmp/CyberChef.zip" | sha256sum -c - && \
    unzip -o /tmp/CyberChef.zip -d /opt/socore/html/cyberchef && \
    mv /opt/socore/html/cyberchef/CyberChef_v${CYBERCHEF_VERSION}.html /opt/socore/html/cyberchef/index.html && \
    rm -f /tmp/CyberChef.zip

EXPOSE 80
EXPOSE 443
