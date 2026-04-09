# Netdef Types And YAML Mapping

This page documents Netplan net definition (netdef) types and how YAML section names map to internal type values.

## Where netdef type is stored

Each parsed netdef is represented by a `NetplanNetDefinition` object.
Its kind is stored in the `type` field:

- `NetplanNetDefinition.type` (`NetplanDefType`)

See:

- `src/abi.h` (`struct netplan_net_definition`)
- `include/types.h` (`typedef enum NetplanDefType`)

## YAML section name -> internal type mapping

The parser maps top-level `network:` device sections to `NetplanDefType` values.
The canonical string names are provided by `netplan_def_type_name()` and defined in `src/names.c`.

| YAML section name | Internal type (`NetplanDefType`) | Notes |
| --- | --- | --- |
| `ethernets` | `NETPLAN_DEF_TYPE_ETHERNET` | Physical Ethernet devices |
| `wifis` | `NETPLAN_DEF_TYPE_WIFI` | Wi-Fi devices |
| `modems` | `NETPLAN_DEF_TYPE_MODEM` | Cellular modem devices |
| `bridges` | `NETPLAN_DEF_TYPE_BRIDGE` | Virtual bridge devices |
| `bonds` | `NETPLAN_DEF_TYPE_BOND` | Bond devices |
| `vlans` | `NETPLAN_DEF_TYPE_VLAN` | VLAN devices |
| `tunnels` | `NETPLAN_DEF_TYPE_TUNNEL` | Tunnel devices (includes WireGuard/VXLAN modes) |
| `vrfs` | `NETPLAN_DEF_TYPE_VRF` | VRF devices |
| `dummy-devices` | `NETPLAN_DEF_TYPE_DUMMY` | Dummy interfaces |
| `virtual-ethernets` | `NETPLAN_DEF_TYPE_VETH` | Veth pairs |
| `nm-devices` | `NETPLAN_DEF_TYPE_NM` | NetworkManager passthrough device type |

## Internal-only and special types

These types exist in the enum but are not normal user-facing YAML section names.

| Internal type | String name | Purpose |
| --- | --- | --- |
| `NETPLAN_DEF_TYPE_NONE` | `NULL` | Sentinel/no type |
| `NETPLAN_DEF_TYPE_VIRTUAL` | no standalone mapping | Marker value used as base for virtual types (`BRIDGE` alias) |
| `NETPLAN_DEF_TYPE_PORT` | `_ovs-ports` | Internal Open vSwitch patch/port netdefs |
| `NETPLAN_DEF_TYPE_NM_PLACEHOLDER_` | no user-facing mapping | Placeholder used to satisfy missing links in specific NetworkManager flows |
| `NETPLAN_DEF_TYPE_MAX_` | `NULL` | Enum upper bound sentinel |

## Enum overview

`NetplanDefType` values are defined in `include/types.h`.
Important details:

- Physical types: `ETHERNET`, `WIFI`, `MODEM`
- Virtual types start at `NETPLAN_DEF_TYPE_VIRTUAL`
- `NETPLAN_DEF_TYPE_BRIDGE = NETPLAN_DEF_TYPE_VIRTUAL` (alias)
- Placeholder and max sentinel are kept at the end of the enum

## Name conversion helpers

Netplan provides helper functions in `src/names.c`:

- `netplan_def_type_name(NetplanDefType)`
  - Converts enum value -> YAML section string (when available)
- `netplan_def_type_from_name(const char*)`
  - Converts section string -> enum value

These use exact string comparison (`g_strcmp0`), so names are case-sensitive.

## Parsing source of truth

The section-to-type mapping is wired in parser handler tables in `src/parse.c` (`network_handlers`), where each section key (for example `"ethernets"`) is tied to `handle_network_type` plus a specific `NetplanDefType` value.
