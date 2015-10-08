# This is a Docker file for alljoyn-deamon and gateway agent (see https://wiki.allseenalliance.org/gateway/getting_started)
FROM bmorin/alljoyn-gwagent:latest

COPY . /config

WORKDIR /alljoyn

RUN mkdir staging && cd staging && cp /config/installPackage.sh . && cp /config/dummyApp1.tar.gz . && sed -i -e 's/mkdir "$tmpDir" || exit 7/mkdir -p "$tmpDir" || exit 7/' -e 's/id -u "$connectorId" &> \/dev\/null/id -u "$connectorId" 2> \/dev\/null/' -e 's/if \[ $? == 0 \]/if \[ $? -eq 0 \]/' -e 's/if \[ $? != 0 \]/if \[ $? -ne 0 \]/' installPackage.sh

WORKDIR /alljoyn/staging

RUN chmod +x installPackage.sh && ./installPackage.sh dummyApp1.tar.gz && cp -r /opt/alljoyn/apps /opt/alljoyn/alljoyn-daemon.d && mv /config/all.acl /opt/alljoyn/apps/dummyapp1/acls/all

# Start 
CMD ifconfig eth0 broadcast 172.17.255.255 up && service ssh start && service alljoyn start && sleep 2 && service alljoyn-gwagent start && sleep 2 && service alljoyn status && service alljoyn-gwagent status && ps -ef | grep "alljoyn-" && ps -ef | grep gwconn && tail -F /var/log/alljoyn-gwagent.log
