# Copyright (c) 2022 Robert Bosch GmbH
#
# This program and the accompanying materials are made available under the
# terms of the Apache License, Version 2.0 which is available at
# https://www.apache.org/licenses/LICENSE-2.0.
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS, WITHOUT
# WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied. See the
# License for the specific language governing permissions and limitations
# under the License.
#
# SPDX-License-Identifier: Apache-2.0

FROM --platform=$BUILDPLATFORM ubuntu:22.04 as builder

RUN apt-get update && apt-get upgrade -y 

# install devcontainer feature prerequisites
RUN apt-get install -y apt-utils openssh-client gnupg2 dirmngr iproute2 \
        procps lsof htop net-tools psmisc manpages manpages-dev \
        curl tree wget rsync ca-certificates unzip init-system-helpers \
        bzip2 zip nano vim-tiny less jq lsb-release git zsh pigz \
        apt-transport-https dialog libc6 libgcc1 libkrb5-3 iptables \
        libgssapi-krb5-2 libicu[0-9][0-9] liblttng-ust[0-9] \
        libstdc++6 zlib1g locales sudo ncdu man-db strace gnupg2 \
        build-essential python3-dev build-essential cmake cppcheck valgrind clang lldb llvm gdb \
        gcc-aarch64-linux-gnu g++-aarch64-linux-gnu binutils-aarch64-linux-gnu

# clean-up
RUN apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

ADD https://raw.githubusercontent.com/devcontainers/features/main/src/common-utils/install.sh /container/scripts/install-common.sh
RUN UID="4000" bash /container/scripts/install-common.sh 
ADD https://raw.githubusercontent.com/devcontainers/features/main/src/docker-in-docker/install.sh /container/scripts/install-dind.sh
RUN VERSION="latest" bash /container/scripts/install-dind.sh

ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/conanfile.py /container/cpp/

# Get all profiles from the SDK
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_aarch64_debug   /container/cpp/.conan/profiles/linux_arm64_debug
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_aarch64_debug   /container/cpp/.conan/profiles/
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_aarch64_release /container/cpp/.conan/profiles/
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_x86_64_debug    /container/cpp/.conan/profiles/linux_amd64_debug
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_x86_64_debug    /container/cpp/.conan/profiles/
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/.conan/profiles/linux_x86_64_release  /container/cpp/.conan/profiles/
ADD --chown=4000 https://raw.githubusercontent.com/SoftwareDefinedVehicle/vehicle-app-cpp-sdk/cross-compilation/install_dependencies.sh           /container/cpp/

COPY ./common/ /scripts/

WORKDIR /scripts
RUN ./install-python.sh
#RUN ./install-dapr-cli.sh
#RUN ./install-k3d-prerequisites.sh
#RUN ./install-k3d-tooling.sh

RUN pip3 install gcovr conan pre-commit cpplint

WORKDIR /container/cpp
USER 4000
WORKDIR /container/cpp/build
WORKDIR /container/cpp

ARG TARGETARCH
RUN bash ./install_dependencies.sh -x $TARGETARCH


# final image
FROM --platform=$TARGETPLATFORM ubuntu:22.04 as final

RUN apt-get update && apt-get upgrade -y 

# install devcontainer feature prerequisites
RUN apt-get install -y apt-utils openssh-client gnupg2 dirmngr iproute2 \
        procps lsof htop net-tools psmisc manpages manpages-dev \
        curl tree wget rsync ca-certificates unzip init-system-helpers \
        bzip2 zip nano vim-tiny less jq lsb-release git zsh pigz \
        apt-transport-https dialog libc6 libgcc1 libkrb5-3 iptables \
        libgssapi-krb5-2 libicu[0-9][0-9] liblttng-ust[0-9] \
        libstdc++6 zlib1g locales sudo ncdu man-db strace gnupg2 \
        build-essential python3-dev build-essential cmake cppcheck valgrind clang lldb llvm gdb

# install cpp packages
RUN apt-get install -y cppcheck clang-format-14 clang-tidy-14

# clean-up
RUN apt-get autoremove -y && apt-get clean -y && rm -rf /var/lib/apt/lists/*

ADD https://raw.githubusercontent.com/devcontainers/features/main/src/common-utils/install.sh /container/scripts/install-common.sh
RUN UID="4000" bash /container/scripts/install-common.sh 
ADD https://raw.githubusercontent.com/devcontainers/features/main/src/docker-in-docker/install.sh /container/scripts/install-dind.sh
RUN VERSION="latest" bash /container/scripts/install-dind.sh

COPY ./common/ /scripts/

WORKDIR /scripts
RUN ./install-python.sh
RUN ./install-dapr-cli.sh
RUN ./install-k3d-prerequisites.sh
RUN ./install-k3d-tooling.sh

RUN pip3 install gcovr conan pre-commit cpplint

COPY --from=builder /home/vscode/.conan /home/vscode/.conan