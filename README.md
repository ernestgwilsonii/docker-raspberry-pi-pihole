### Pi-hole https://github.com/pi-hole/docker-pi-hole/

```
# Create bind mounted directories
# Pi-hole https://github.com/pi-hole/docker-pi-hole/
mkdir -p /opt/docker/pi-hole/etc/pihole
mkdir -p /opt/docker/pi-hole/etc/dnsmasq.d
chmod -R a+rw /opt/docker/pi-hole
chown -R pi:docker /opt/docker/pi-hole

# Deploy the stack into a Docker Swarm
docker stack deploy -c docker-compose.yml pi-hole
#docker stack rm pi-hole

# Verify
docker stack ls
docker service ls | grep pi-hole
docker service logs -f pi-hole_pihole
docker ps
```

### macvlan notes and testing and nnotes

```
# Non-SWARM Mode
# REF: https://forums.raspberrypi.com/viewtopic.php?t=329272

docker network ls

# Create macvlan network for Pi-hole
docker network create -d macvlan \
	--subnet=172.28.0.0/24 \
	--gateway=172.28.0.1 \
	--ip-range=172.28.0.2/32 \
	-o parent=eth0 \
	-o macvlan_mode=bridge \
pi-hole_network

docker network ls | grep macvlan

# Run Pi-hole on Docker
docker run -d \
    --rm \
    --name pi-hole \
    --hostname pi-hole \
    --domainname landenberg.local \
    --net pi-hole_network \
    --ip 172.28.0.2 \
pihole/pihole:latest



# Docker SWARM Mode
# REF: https://jpft.win/docker-swarm-macvlan/

# Create network on all SWARM nodes first!
docker network create --config-only --subnet 172.28.0.0/24 -o parent=eth0 --gateway=172.28.0.1 --ip-range 172.28.0.2/32 pi-hole-network-config

# Run this once on a SWARM Manager node
docker network create -d macvlan --scope swarm --attachable --config-from pi-hole-network-config pi-hole_network
docker network inspect pi-hole-network-config
docker network inspect pi-hole_network

# Launch service on SWARM
docker service create --replicas 1 --name pi-hole --network pi-hole_network pihole/pihole:latest
```