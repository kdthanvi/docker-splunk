# Copyright 2018 Splunk
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
FROM base-debian-9:latest

ARG SPLUNK_FILENAME
ARG SPLUNK_BUILD_URL
ARG SPLUNK_DEFAULTS_URL

ENV SPLUNK_HOME=/opt/splunkforwarder \
    SPLUNK_GROUP=splunk \
    SPLUNK_GROUP_ID=9999 \
    SPLUNK_USER=splunk \
    SPLUNK_USER_ID=9999 \
    SPLUNK_ROLE=splunk_universal_forwarder \
    SPLUNK_FILENAME=${SPLUNK_FILENAME} \
    SPLUNK_DEFAULTS_URL=${SPLUNK_DEFAULTS_URL} \
    SPLUNK_ANSIBLE_HOME=/opt/ansible \
    SPLUNK_OPT=/opt

# Setup users and download Splunk
RUN groupadd -r -g ${SPLUNK_GROUP_ID} ${SPLUNK_GROUP} \
    && useradd -r -m -g ${SPLUNK_GROUP} -u ${SPLUNK_USER_ID} ${SPLUNK_USER} \
    && usermod -aG sudo ${SPLUNK_USER} \
    && sed -i -e 's/%sudo\s\+ALL=(ALL\(:ALL\)\?)\s\+ALL/%sudo ALL=NOPASSWD:ALL/g' /etc/sudoers \
    && echo "Downloading Splunk and validating the checksum at: ${SPLUNK_BUILD_URL}" \
    && wget -qO /tmp/${SPLUNK_FILENAME} ${SPLUNK_BUILD_URL} \
    && wget -qO /tmp/${SPLUNK_FILENAME}.sha512 ${SPLUNK_BUILD_URL}.sha512 \
    && (cd /tmp && sha512sum -c ${SPLUNK_FILENAME}.sha512) \
    && mv /tmp/${SPLUNK_FILENAME} /tmp/splunk.tgz \
    && rm -rf /tmp/${SPLUNK_FILENAME}.sha512

USER ${SPLUNK_USER_ID}
COPY splunk-ansible ${SPLUNK_ANSIBLE_HOME}
COPY [ "uf/debian-9/entrypoint.sh", "uf/debian-9/checkstate.sh", "uf/debian-9/createdefaults.py", "/sbin/"]
COPY [ "uf/debian-9/bin/ta-dockerlogs_fileinput.tar.gz", "uf/debian-9/bin/ta-dockerstats.tar.gz", "/tmp/" ]

EXPOSE 8089 8088 9997
VOLUME [ "/opt/splunkforwarder/etc", "/opt/splunkforwarder/var" ]

HEALTHCHECK --interval=30s --timeout=30s --start-period=3m --retries=5 CMD /sbin/checkstate.sh || exit 1

ENTRYPOINT [ "/sbin/entrypoint.sh" ]
CMD [ "start-service" ]
