# This is a Docker file for alljoyn-deamon and gateway agent (see https://wiki.allseenalliance.org/gateway/getting_started)
FROM ubuntu:latest

# Install all necessary packages
RUN apt-get update && apt-get -f -y install --no-install-recommends build-essential libgtk2.0-dev libssl-dev xsltproc libxml2-dev libcap-dev gcc g++ python scons git-core && apt-get clean && rm -rf /var/lib/apt/lists/* 

COPY . /config

ENV GIT_SSL_NO_VERIFY true

# Enable AllJoyn logging
ENV ER_DEBUG_ALL 0xFFFF

# Get AllJoyn and Gateway Agent sources, v14.12, compile and install them, and clean afterwards (3.3GB saved!)
WORKDIR /alljoyn
RUN mkdir -p core && cd core && git clone https://git.allseenalliance.org/gerrit/core/alljoyn.git && cd alljoyn && git checkout v14.12 && scons BINDINGS=cpp OS=linux CPU=x86_64 VARIANT=debug BUILD_SERVICES_SAMPLES=off POLICYDB=on WS=off && mkdir -p /opt/alljoyn/alljoyn-daemon.d && cp build/linux/x86_64/debug/dist/cpp/bin/alljoyn-daemon /usr/bin/ && find build -name "*\.so" -exec cp {} /usr/lib/ \; && cd ../.. && mkdir services && mkdir gateway && cd services && git clone https://git.allseenalliance.org/gerrit/services/base.git && cd base && git checkout v14.12 && cd ../../gateway && git clone https://git.allseenalliance.org/gerrit/gateway/gwagent.git && cd gwagent && scons BINDINGS=cpp OS=linux CPU=x86_64 VARIANT=debug BUILD_SERVICES_SAMPLES=off POLICYDB=on ALLJOYN_DISTDIR=/alljoyn/core/alljoyn/build/linux/x86_64/debug/dist WS=off && find build -name "*\.so" -exec cp {} /usr/lib \; && find build -name "alljoyn-gwagent" -type f -exec cp {} /usr/bin/alljoyn-gwagent \; && mkdir /opt/alljoyn/gwagent && cp /alljoyn/gateway/gwagent/cpp/GatewayMgmtApp/manifest.xsd /opt/alljoyn/gwagent/ && chmod a+x /config/alljoyn-gwagent.init && cp /config/alljoyn-gwagent.init /etc/init.d/alljoyn-gwagent && rm -rf /alljoyn/gateway && rm -rf /alljoyn/core 

# Configure
RUN mv /config/config.xml /opt/alljoyn/alljoyn-daemon.d/config.xml && chmod a+x /config/alljoyn.init && mv /config/alljoyn.init /etc/init.d/alljoyn

# Install and Configure ssh (for debug purpose)
ENV NOTVISIBLE "in users profile"
RUN apt-get update && apt-get -f -y install --no-install-recommends openssh-server && apt-get clean && rm -rf /var/lib/apt/lists/* && mkdir /var/run/sshd && echo 'root:screencast' | chpasswd && sed -i 's/PermitRootLogin without-password/PermitRootLogin yes/' /etc/ssh/sshd_config && sed 's@session\s*required\s*pam_loginuid.so@session optional pam_loginuid.so@g' -i /etc/pam.d/sshd && echo "export VISIBLE=now" >> /etc/profile

# SSH port
EXPOSE 22

# Expose Contact port for AllJoyn standard (TCP) and multiplexed constrained (UDP) messaging
EXPOSE 9955/udp 
EXPOSE 9955/tcp
# Expose AllJoyn Name service port
EXPOSE 9956/udp
