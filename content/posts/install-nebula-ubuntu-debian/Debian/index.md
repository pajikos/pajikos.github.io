---
title: "Install nebula on Ubuntu/Debian"
date: "2022-01-24"
categories: 
  - "network"
tags: 
  - "nebula"
  - "cloud"
draft: true
---
{{< admonition note "Nebula Network" >}}
[Nebula](https://github.com/slackhq/nebula) is a scalable overlay networking tool with a focus on performance, simplicity, and security. It lets you seamlessly connect computers anywhere in the world.
{{< /admonition >}}



## Network and devices specification
./nebula-cert sign -name "oraclevm1" -ip "172.16.88.5/24" -groups "servers,oracle"
scp oraclevm1.* ca.crt ubuntu@193.122.55.60:/tmp


wget https://github.com/slackhq/nebula/releases/download/v1.5.2/nebula-linux-amd64.tar.gz
sudo mkdir /opt/nebula
sudo mkdir /opt/nebula/certs
sudo tar -C /opt/nebula -xvf nebula-linux-amd64.tar.gz

ssh -t ubuntu@193.122.55.60 -C "sudo mkdir /opt/nebula/certs"
ssh -t ubuntu@193.122.55.60 -C "sudo mv /tmp/oraclevm1.key /opt/nebula/certs/client.key"
ssh -t ubuntu@193.122.55.60 -C "sudo mv /tmp/oraclevm1.crt /opt/nebula/certs/client.crt"
ssh -t ubuntu@193.122.55.60 -C "sudo mv /tmp/ca.crt /opt/nebula/certs"

scp config.yml.oraclevm1 ubuntu@193.122.55.60:/opt/nebula/config.yaml

sudo ufw allow 4242/udp


scp nebula-start.sh nebula-start.service ubuntu@193.122.55.60:/tmp

ssh -t ubuntu@193.122.55.60 -C "sudo -- sh -c 'mv /tmp/nebula-start.sh /usr/bin/nebula-start.sh && chmod +x /usr/bin/nebula-start.sh'"

ssh -t ubuntu@193.122.55.60 -C "sudo -- sh -c 'mv /tmp/nebula-start.service /etc/systemd/system/nebula-start.service && chmod 644 /etc/systemd/system/nebula-start.service'"


ssh -t ubuntu@193.122.55.60 -C "sudo -- sh -c 'systemctl daemon-reload && sleep 5 && systemctl enable nebula-start.service && systemctl start nebula-start.service && systemctl status nebula-start.service'"


