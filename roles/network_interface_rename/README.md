# Ansible Role: network-interface-rename

## Description

This Ansible role standardizes network interface naming across servers by renaming the primary network interface to a consistent name (default: `net0`). This ensures predictable network interface naming across your infrastructure, which can simplify configuration management and improve reliability.

The role uses Netplan to handle the interface renaming process on Ubuntu systems. It automatically identifies the primary interface, backs up existing configurations, and creates a new configuration with the standardized interface name.

## Requirements

- Ubuntu 18.04+ or other Netplan-compatible distribution
- Ansible 2.9+
- Root access on target hosts

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `new_interface_name` | `net0` | The standardized name to apply to the primary network interface |

## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: all
  become: true
  roles:
    - role: network-interface-rename
      tags: interface
```

With custom interface name:

```yaml
---
- hosts: all
  become: true
  roles:
    - role: network-interface-rename
      tags: interface
      vars:
        new_interface_name: "eth0"
```

## How It Works

1. Checks if the target interface name already exists on the system
2. If the interface doesn't exist:
   - Identifies the current primary interface and its MAC address
   - Backs up existing netplan configurations
   - Creates a new netplan configuration that renames the interface based on MAC address
   - Applies the new configuration

## Verification

After the role executes successfully, you can verify the implementation with:

```bash
# Check interface name
ip a

# Verify netplan configuration
cat /etc/netplan/99-net0-cfg.yaml
```

The role also outputs detailed information about the renamed interface at the end of execution.

## Testing

To test this role in isolation before deploying to production:

1. Run the role with `--check` mode first to preview changes
2. Apply the role and verify network connectivity is maintained
3. Reboot the system to verify persistence

## Limitations and Considerations

- This role is designed for systems using Netplan as the network configuration tool
- Running this role may cause momentary network interruptions
- Use caution when applying to remote systems, as network connectivity may be affected
- Multi-homed systems (with multiple network interfaces) requires additional modifications