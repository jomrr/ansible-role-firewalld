# Ansible Role: firewalld

![GitHub](https://img.shields.io/github/license/jomrr/ansible-role-firewalld)
![GitHub last commit](https://img.shields.io/github/last-commit/jomrr/ansible-role-firewalld)
![GitHub issues](https://img.shields.io/github/issues-raw/jomrr/ansible-role-firewalld)
[![dev](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-firewalld/dev.yml?branch=dev&label=dev)](https://github.com/jomrr/ansible-role-firewalld/actions/workflows/dev.yml?query=branch%3Adev)
[![main](https://img.shields.io/github/actions/workflow/status/jomrr/ansible-role-firewalld/main.yml?branch=main&label=main)](https://github.com/jomrr/ansible-role-firewalld/actions/workflows/main.yml?query=branch%3Amain)

Ansible role for setting up firewalld.

## Purpose

Install, configure, enable, and start firewalld. Manage permanent zones,
services, IPsets, policies, and direct rules, and reload changed definitions.
Repeated runs with unchanged inputs are idempotent.

## Scope

### Managed

- firewalld configuration and named XML definitions.
- Service enablement, startup, and reload after configuration changes.

### Not Managed

- Other firewall services and NetworkManager connection profiles.
- Removal of unlisted zone, service, policy, or IPset files.

## Requirements

- Root privileges through become and gathered network facts.
- A host with firewalld support and the kernel facilities required by the
  selected rules.
- Direct rules require the iptables utilities for the selected address family.

## Dependencies

```yaml
collections:
  - name: community.general
    version: '>=12.0.0'
```

## Role Variables

### `firewalld_backup`

Type: `bool`. Required: `false`.

Back up configuration files before replacing them.

Default:

```yaml
firewalld_backup: true
```

### `firewalld_default_zone`

Type: `str`. Required: `false`.

Zone used for connections without an explicit zone assignment.

Default:

```yaml
firewalld_default_zone: public
```

### `firewalld_cleanup_on_exit`

Type: `str`. Required: `false`.

Remove firewall rules when the daemon stops.

Default:

```yaml
firewalld_cleanup_on_exit: 'yes'
```

### `firewalld_cleanup_modules_on_exit`

Type: `str`. Required: `false`.

Unload firewall kernel modules when the daemon stops.

Default:

```yaml
firewalld_cleanup_modules_on_exit: 'yes'
```

### `firewalld_lockdown`

Type: `str`. Required: `false`.

Restrict D-Bus changes to the lockdown whitelist on versions supporting
lockdown.

Default:

```yaml
firewalld_lockdown: 'no'
```

### `firewalld_ipv6_rpfilter`

Type: `str`. Required: `false`.

IPv6 reverse-path filtering mode.

Default:

```yaml
firewalld_ipv6_rpfilter: 'yes'
```

### `firewalld_individual_calls`

Type: `str`. Required: `false`.

Use individual rule calls on versions supporting this option.

Default:

```yaml
firewalld_individual_calls: 'no'
```

### `firewalld_log_denied`

Type: `str`. Required: `false`.

Packet classes logged before rejection or dropping.

Default:

```yaml
firewalld_log_denied: 'off'
```

### `firewalld_firewall_backend`

Type: `str`. Required: `false`.

Backend used by firewalld.

Default:

```yaml
firewalld_firewall_backend: nftables
```

### `firewalld_flush_all_on_reload`

Type: `str`. Required: `false`.

Flush runtime rules when permanent configuration is reloaded.

Default:

```yaml
firewalld_flush_all_on_reload: 'yes'
```

### `firewalld_rfc3964_ipv4`

Type: `str`. Required: `false`.

Filter IPv6 6to4 destinations containing non-public IPv4 addresses.

Default:

```yaml
firewalld_rfc3964_ipv4: 'yes'
```

### `firewalld_nftables_flowtable`

Type: `str`. Required: `false`.

Space-separated flowtable interfaces, or off to disable flowtables.

Default:

```yaml
firewalld_nftables_flowtable: 'off'
```

### `firewalld_nftables_counters`

Type: `str`. Required: `false`.

Add counters to nftables rules.

Default:

```yaml
firewalld_nftables_counters: 'no'
```

### `firewalld_policy_target`

Type: `str`. Required: `false`.

Default action for traffic not matched by a policy rule.

Default:

```yaml
firewalld_policy_target: CONTINUE
```

### `firewalld_direct_rules`

Type: `list`. Required: `false`.

Complete direct rule list; an empty list clears direct.xml.

Default:

```yaml
firewalld_direct_rules: []
```

### `firewalld_ipsets`

Type: `list`. Required: `false`.

IPset definitions written under /etc/firewalld/ipsets.

Default:

```yaml
firewalld_ipsets: []
```

### `firewalld_policies`

Type: `list`. Required: `false`.

Policy definitions with optional per-policy target overrides.

Default:

```yaml
firewalld_policies: []
```

### `firewalld_services`

Type: `list`. Required: `false`.

Custom service definitions and their ports.

Default:

```yaml
firewalld_services: []
```

### `firewalld_zones`

Type: `list`. Required: `false`.

Zone definitions; unlisted definition files are retained.

Default:

```yaml
firewalld_zones:
  - name: '{{ firewalld_default_zone }}'
    description: For use in public areas. You do not trust the other computers on
      networks to not harm your computer. Only selected incoming connections are
      accepted.
    forward_ports: []
    icmp_block_inversion: false
    icmp_blocks: []
    interfaces:
      - '{{ ansible_facts.default_ipv4.interface }}'
    masquerade: false
    ports: []
    rules: []
    services:
      - ssh
    source_addresses: []
    source_ports: []
    target: default
    tcp_mss_clamp: 0
```

## Managed Files

- `/etc/firewalld/firewalld.conf`
- `/etc/firewalld/direct.xml`
- `/etc/firewalld/{ipsets,policies,services,zones}/<name>.xml`

## Check Mode

Package, file, template, and service tasks support check mode.

- The native configuration check is skipped in check mode because candidate
  files are not written.
- Check mode on a fresh host cannot validate service state before firewalld is
  installed.

## Service Behavior

Check the complete permanent configuration before starting the service; reload
changed files through a handler.

### Handlers

- Reload firewalld after configuration changes.

## Security Notes

- The default public zone permits SSH. HTTP and HTTPS require explicit service
  entries.
- Policies default to CONTINUE; an explicit target can override
  firewalld_policy_target for each policy.
- Configuration files are owned by root with mode 0640; definition directories
  use mode 0750.

## Operational Notes

- An empty firewalld_direct_rules list clears direct.xml, including rules
  previously written by the role.
- Each named XML definition is owned by this role; external permanent changes to
  the same file are overwritten.
- Optional object attributes may be omitted. Services require name and ports;
  policies require name, ingress_zone, and egress_zone; IPsets require name and
  type.
- Rich-rule action.type selects accept, reject, drop, or mark.
  action.reject_type optionally selects the ICMP rejection type; action.mark
  supplies the mark value.
- firewall-offline-cmd --check-config checks the complete configuration after
  files are written. It does not accept an individual template candidate file.
  Failed validation stops the run before the role starts or reloads firewalld;
  backups retain previous files.
- Legacy daemon settings remain available for older firewalld releases. Support
  for lockdown and backend-specific settings depends on the installed release.

## Supported Platforms

| OS Family | Distribution | Version | Container Image |
| --------- | ------------ | ------- | --------------- |
| RedHat | AlmaLinux | latest | [jomrr/molecule-almalinux:latest](https://hub.docker.com/r/jomrr/molecule-almalinux) |
| Debian | Debian | latest | [jomrr/molecule-debian:latest](https://hub.docker.com/r/jomrr/molecule-debian) |
| RedHat | Fedora | latest | [jomrr/molecule-fedora:latest](https://hub.docker.com/r/jomrr/molecule-fedora) |
| Suse | OpenSuse Leap | latest | [jomrr/molecule-opensuse-leap:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-leap) |
| Suse | OpenSuse Tumbleweed | latest | [jomrr/molecule-opensuse-tumbleweed:latest](https://hub.docker.com/r/jomrr/molecule-opensuse-tumbleweed) |
| Debian | Ubuntu | latest | [jomrr/molecule-ubuntu:latest](https://hub.docker.com/r/jomrr/molecule-ubuntu) |

## Example Playbook

### Allow a web service

Add HTTP and HTTPS to the public zone explicitly.

```yaml
---
- name: Configure firewall
  hosts: all
  gather_facts: true
  roles:
    - role: jomrr.firewalld
      firewalld_zones:
        - name: public
          services: [ssh, http, https]
```

## References

- [firewalld](https://firewalld.org/)
- [Configuration checking](https://firewalld.org/documentation/man-pages/firewall-offline-cmd.html)

## Author

[Jonas Mauer](https://github.com/jomrr)

## License

This project is licensed under the MIT License.
See [LICENSE](LICENSE) for the full license text.

Copyright (c) 2024 Jonas Mauer.
