# Network Simulator for QUIC benchmarking

The build script builds the network simulator (as found in the [sim](sim) directory) and a client and a server (as found in the [endpoint](endpoint) directory).

## Network topology

The build script creates two networks, `leftnet` (192.168.0.0/24) and `rightnet` (192.168.100.0/24). Leftnet is connected to the client, and rightnet is connected to the server. The ns3 simulation sits in the middle and forwards packets from leftnet to rightnet and vice versa, through the ns3 simulation.

```
                                      |‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|
                                      |                      sim                         |
                                      |                                                  |      
|‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|     |‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|‾‾‾‾‾‾‾‾‾‾‾‾‾|     |‾‾‾‾‾‾‾‾|     |‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|     |‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾‾|
|     client    |     | docker-bridge |     eth0    |     |        |     |     eth1      | docker-bridge |     |      server     |
|               |-----|               |             |-----|  ns-3  |-----|               |               |-----|                 |
| 192.168.0.100 |     |  192.168.0.1  | 192.168.0.2 |     |        |     | 192.168.100.2 | 192.168.100.1 |     | 192.168.100.100 |
|_______________|     |_______________|_____________|     |________|     |_______________|_______________|     |_________________|
                                      |                                                  |
                                      |__________________________________________________|
```

## Running a simulation

### Setting up the networks

To set up `leftnet` and `rightnet`, run the following commands
```bash
docker network create leftnet --subnet 192.168.0.0/24
docker network create rightnet --subnet 192.168.100.0/24
```

The networks can be removed by executing
```bash
docker network rm leftnet rightnet
```

### Running the simulator

To run the simulator, you need to set up the networks first, as described above.
After that, build and run the simulator:
```bash
./run.sh "simple-p2p --delay=10ms --bandwidth=10Mbps"
```

All paramters to `run.sh` are passed to waf, i.e. the command run inside the container will be `./waf --run "simple-p2p --delay=10ms --bandwidth=10Mbps"`.

### Running the endpoints

The [endpoint](endpoint) directory contains the base docker image for an endpoint container. The built image is available on [dockerhub](https://hub.docker.com/r/martenseemann/quic-network-simulator-endpoint).

If you want to build the endpoint image, you can do so by running
```bash
docker build endpoint/ -t endpoint
```

Run the client:
```bash
docker run --cap-add=NET_ADMIN --network leftnet --ip 192.168.0.100 -it --entrypoint /bin/bash endpoint 
```

And the server:
```bash
docker run --cap-add=NET_ADMIN --network rightnet --ip 192.168.100.100 -it --entrypoint /bin/bash endpoint 
```

Note that in order to configure the containers to work in this simulation, you have to set up the routing by running (inside the container):
```
./setup
```
