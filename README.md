# Description

This role provisions a [Nimbus](https://nimbus.team/) installation that can act as an Eth2 network bootstrap node.

# Introduction

The role will:

* Checkout a branch from the [nimbus-eth2](https://github.com/status-im/nimbus-eth2) repo
* Build it using the [`build.sh`](./templates/build.sh.j2) Bash script
* Schedule regular builds using [Scheduled Tasks](https://en.wikipedia.org/wiki/Windows_Task_Scheduler)
* Start a node by defining a [Windows Service](https://en.wikipedia.org/wiki/Windows_service) using [WinSW](https://github.com/winsw/winsw)

# Ports

The service exposes three ports by default:

* `9000` - LibP2P peering port. Must __ALWAYS__ be public.
* `9200` - JSON RPC port. Must __NEVER__ be public.
* `9900` - Prometheus metrics port. Should not be public.

# Installation

Add to your `requirements.yml` file:
```yaml
- name: infra-role-beacon-node-windows
  src: git@github.com:status-im/infra-role-beacon-node-windows.git
  scm: git
```

# Configuration

Minimum configuration would include.
```yaml
# branch which should be built
beacon_node_repo_branch: 'stable'
# ethereum network to connect to
beacon_node_network: 'mainnet'
# optional setting for debug mode
beacon_node_log_level: 'DEBUG'
# WebSocket or HTTP URLs for execution layet Engine API
beacon_node_exec_layer_urls:
  - 'wss://erigon.internal.example.org:8551'
  - 'http://geth.internal.example.org:8552'
```
The order of Web Socket URLs matters. First is the default, the rest are fallbacks.

Most non-sensitive configuration resides in `conf/config.toml` file in service directory.

# Management

## Service

The services are managed using [WinSW](https://github.com/winsw/winsw).
```
admin@windows-01 MINGW64 .../nimbus/beacon-node-prater-stable
$ ls -l
total 665
-rwxr-xr-x 1 admin 197121 655872 Jul 16 14:37 beacon-node-prater-stable.exe*
-rw-r--r-- 1 admin 197121   1358 Oct 26 09:02 beacon-node-prater-stable.yml
drwxr-xr-x 1 admin 197121      0 Oct 21 13:23 bin/
-rwxr-xr-x 1 admin 197121   2639 Oct  7 07:51 build.sh*
drwxr-xr-x 1 admin 197121      0 Jul 16 14:49 data/
drwxr-xr-x 1 admin 197121      0 Oct 26 09:03 logs/
drwxr-xr-x 1 admin 197121      0 Oct 21 13:03 repo/
-rwxr-xr-x 1 admin 197121    682 Oct  7 07:51 rpc.sh*

admin@windows-01 MINGW64 .../nimbus/beacon-node-prater-stable
$ ./beacon-node-prater-stable.exe status
Started

admin@windows-01 MINGW64 .../nimbus/beacon-node-prater-stable
$ ./beacon-node-prater-stable.exe restart
2021-09-29 17:33:27,563 INFO  - Stopping service 'Beacon Node prater (stale) (beacon-node-prater-stable)'...
2021-09-29 17:33:27,825 INFO  - Starting service 'Beacon Node prater (stale) (beacon-node-prater-stable)'...
2021-09-29 17:33:28,366 INFO  - Service 'Beacon Node prater (stale) (beacon-node-prater-stable)' restarted successfully.
```

## Builds

Builds are performed via a Bash `build.sh` script located in the service directory.
```
admin@windows-01 MINGW64 .../nimbus/beacon-node-prater-stable
$ ./build.sh
 >>> Fetching changes...
HEAD is now at 9e8081e4 Merge branch 'stable' into unstable
 >>> Binaries already built.
 >>> No binaries required update.
 >>> SUCCESS!
```
The scheduled tasks that trigger node builds can be checked with `Get-ScheduledTask`:
```log
PS C:\Users\admin> Get-ScheduledTask -TaskName 'build-beacon-node*' | ft State,TaskName,Description,Date

State TaskName                          Description                                  Date
----- --------                          -----------                                  ----
Ready build-beacon-node-prater-stable   Daily rebuild of Nimbus beacon node binaries 2021-09-29T13:00:00
Ready build-beacon-node-prater-testing  Daily rebuild of Nimbus beacon node binaries 2021-09-29T17:00:00
Ready build-beacon-node-prater-unstable Daily rebuild of Nimbus beacon node binaries 2021-09-29T15:00:00
```
The task can be started using [`Start-ScheduledTask`](https://docs.microsoft.com/en-us/powershell/module/scheduledtasks/start-scheduledtask) and its logs can be looked up using [`Get-WinEvent`](https://docs.microsoft.com/en-us/powershell/module/microsoft.powershell.diagnostics/get-winevent):
```log
PS C:\Users\nimbus\beacon-node-prater-stable> Get-WinEvent -LogName Microsoft-Windows-TaskScheduler/Operational | Select -First 30 | Where Message -Match '.*stable.*' | Select -First 4 | ft -Property TimeCreated,Message

TimeCreated           Message
-----------           -------
10/26/2021 9:23:19 AM Task Scheduler successfully finished "{d49d3613-f5bc-4714-a95b-e893e4419031}" instance of the "\build-beacon-node-prater-stable" task for user "NT AUTHORITY\SYSTEM".
10/26/2021 9:23:19 AM Task Scheduler successfully completed task "\build-beacon-node-prater-stable" , instance "{d49d3613-f5bc-4714-a95b-e893e4419031}" , action "C:\ProgramData\scoop\apps\git\current\bin\bash.exe" with return code 0.
10/26/2021 9:23:18 AM Task Scheduler launched "{d49d3613-f5bc-4714-a95b-e893e4419031}"  instance of task "\build-beacon-node-prater-stable"  for user "System" .
10/26/2021 9:23:18 AM Task Scheduler launched action "C:\ProgramData\scoop\apps\git\current\bin\bash.exe" in instance "{d49d3613-f5bc-4714-a95b-e893e4419031}" of task "\build-beacon-node-prater-stable".
```

And the logs from the build script itself can be found in the `logs` directory:
```
admin@windows-01 MINGW64 .../nimbus/beacon-node-prater-stable
$ tail -n5 logs/build.log
 >>> Fetching changes...
HEAD is now at 9e8081e4 Merge branch 'stable' into unstable
 >>> Binaries already built.
 >>> No binaries required update.
 >>> SUCCESS!
```

# Known Issues

* The WinSW wrapper fails to grant `Log On As Service` right. ([winsw#840](https://github.com/winsw/winsw/issues/840))
* The `--log-file` flag has no effect on windows. ([nimbus-eth2#2326](https://github.com/status-im/nimbus-eth2/issues/2326))
