# Description

This role provisions a [Nimbus](https://nimbus.team/) installation that can act as an ETH2 network bootstrap node.

# Ports

The service exposes three ports by default:

* `9000` - LibP2P peering port. Must __ALWAYS__ be public.
* `9200` - JSON RPC port. Must __NEVER__ be public.
* `9900` - Prometheus metrics port. Should not be public.

# Configuration

Minimum configuration would include.
```yaml
beacon_node_network: 'testnet0'
# Infura Web Sockets URLs
beacon_node_web3_urls: ['wss://mainnet.infura.io/ws/v3/123qwe123qwe123qwe']
```
The order of Web Socket URLs matters. First is the default, the rest are fallbacks.

It might be useful to increase the log verbosity level:
```yaml
beacon_node_log_level: DEBUG
```

# Management

The containers are managed using [WinSW](https://github.com/winsw/winsw).
```
TODO
```
