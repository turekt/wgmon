# Wireguard connection monitor

Wireguard connection monitor listens and reports newly connected wireguard peers via webhook.

## Project

There are existing Wireguard connection monitors available and they all use a similar approach to monitoring connections.

This repo tries to optimize the monitoring process and do things a bit differently:
- Monitor only when a connection is triggered, instead of constantly running `wg show`
  - utilizes gopacket bpf filter that triggers checks on received packet
- Scan for currently connected peers by interacting directly with kernel
  - utilizes netlink to get info on wireguard links instead of executing `wg show all dump` on operating system level
- Recognize when peer has disconnected
  - after peer connects, initiate ticker that checks connection info and, if packets were not received for some time, register as a disconnect
  - the gap between actual disconnect and recognition can be optimized with usage of `PersistentKeepAlive` and smaller tick period

All this is paired with minimal dependencies needed to run things smoothly.

## Usage

The project is provided and intended to be ran within a container, therefore Docker or Podman installed is the only requirement.
```
# Podman
podman build -t wg .
# Docker
docker build -t wg .
# For specific wireguard interface name
docker build --build-arg intfname=custom -t wg .
```

Wireguard keys and config should be stored in `/etc/wireguard` on host. Project assumes listen port 3000, make sure to adapt project params per your config.

For Podman there is available kube file. Run the project with:
```
podman kube play kube.yaml
```

For Docker there is a docker compose file.
Environment config in compose file:
* `webhook` - URL where info about new connected wireguard peer will be sent
* `interface` - host interface on which wireguard is listening
* `filter` - BPF filter for wireguard traffic; containing protocol and wireguard listening port. default: `udp and dst port 3000`

```
docker compose up
```

Project also runs locally.
```
go build -o app
sudo ./app
```

## Tests

To run tests:
```
# Skips live test
go test -v ./...
# Including live test
go test -v ./... -args -live
```
