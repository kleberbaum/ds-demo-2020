# ds-demo-2020
A demo repository for docker swarm mode.

## KVM server base setup #######################################################

This example is split up into snapshots.

## demobase #######################################################################
    ssh root@0.0.0.0
    ln -sfn /usr/share/zoneinfo/UTC /etc/localtime
    localectl set-locale LANG=en_US.UTF-8
    echo 'glhf' > /etc/hostname
    reboot
    ssh root@0.0.0.0
    apt-get update
    apt-get dist-upgrade
    apt-get install fish
  	adduser username -c 'Full Name' -G users && passwd username
    usermod root -s '/usr/bin/fish'
    sed -i 's/.*PermitRootLogin ...\?$/PermitRootLogin no/g' /etc/ssh/sshd_config
    poweroff

## demodocker #####################################################################
    ssh username@0.0.0.0
    su -
    apt-get install apt-transport-https
    add-apt-repository 'deb [arch=amd64] https://download.docker.com/linux/ubuntu bionic stable'
    wget -qO- 'https://download.docker.com/linux/ubuntu/gpg' | apt-key add –
    apt-get update
    apt-get install docker-ce=5:19.03.7~3-0~ubuntu-bionic
    systemctl start docker.service
    systemctl enable docker.service
    /lib/systemd/systemd-sysv-install enable docker
    poweroff

## demods #########################################################################
    ssh username@0.0.0.0
    su -
    docker swarm init --advertise-addr 0.0.0.0
    apt-get install git
    git clone https://gist.github.com/2b3bfd8ff886015bbce8.git /tmp/docker-shell
    install /tmp/docker-shell/docker-shell /usr/local/sbin/
    rm -rf /tmp/docker-shell
    docker network create --driver overlay --subnet=185.143.7.0/27 traefik-net
    docker service create --name traefik --network traefik-net --publish mode=host,target=80,published=80 --publish 8080:8080 --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock --constraint=node.role==manager traefik:1.7-alpine --docker.swarmMode --docker.domain=snek.at --docker.watch --api --docker
    docker service create --name portainer --network traefik-net --publish 9000:9000 --label "traefik.enable=true" --label "traefik.port=9000" --label "traefik.docker.network=traefik-net" --label "traefik.frontend.rule=Host:p.snek.at" --label "traefik.backend=portainer" --constraint 'node.role == manager' --mount type=bind,src=//var/run/docker.sock,dst=/var/run/docker.sock --mount type=volume,src=portainer_data,dst=/data portainer/portainer:1.23.0 -H unix:///var/run/docker.sock
    poweroff

## demofun ########################################################################
    docker service ls
    docker stack ls
    curl -O https://raw.githubusercontent.com/docker/example-voting-app/master/docker-stack.yml /tmp
    docker stack deploy wetterhost --compose-file /tmp/docker-stack.yml
    poweroff
