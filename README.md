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

The logs for scheduled tasks running builds can be looked up using `Get-WinEvent`:
```
PS C:\Users\nimbus\beacon-node-prater-stable> Get-WinEvent -LogName Microsoft-Windows-TaskScheduler/Operational | Select -First 4 | ft -Property TimeCreated,Message

TimeCreated           Message
-----------           -------
6/17/2021 11:09:46 AM Task Scheduler did not launch task "\build-beacon-node-prater-unstable"  because instance "{afe16e96-be8d-40b8-a398-b3be6513de88}"  of the same task is already running.
6/17/2021 11:09:46 AM Task Scheduler launched "{72498e41-91ee-4318-95f4-54daca2acf2f}"  instance of task "\build-beacon-node-prater-unstable"  for user "System" .
6/17/2021 11:09:46 AM Task Scheduler queued instance "{72498e41-91ee-4318-95f4-54daca2acf2f}"  of task "\build-beacon-node-prater-unstable".
6/17/2021 11:06:55 AM Task Scheduler successfully finished "{747c0708-0908-4514-a4b4-c6e71d6132c0}" instance of the "\build-beacon-node-prater-stable" task for user "NT AUTHORITY\SYSTEM".
```

# Known Issues

* The WinSW wrapper fails to grant `Log On As Service` right. ([winsw#840](https://github.com/winsw/winsw/issues/840))
* The `--log-file` flag has no effect on windows. ([nimbus-eth2#2326](https://github.com/status-im/nimbus-eth2/issues/2326)
