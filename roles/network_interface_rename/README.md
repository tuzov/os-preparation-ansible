# Ansible Role: network_interface_rename

## Description

This Ansible role standardizes network interface naming across servers by renaming the primary network interface to a consistent name (default: `net0`). This ensures predictable network interface naming across your infrastructure, which can simplify configuration management and improve reliability.

The role uses Netplan to handle the interface renaming process on Ubuntu systems. It automatically identifies the primary interface and changes `set-name` of the interface.

## Requirements

- Ubuntu 18.04+ or other Netplan-compatible distribution
- Ansible 2.9+
- Root access on target hosts

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `network_interface_rename__new_interface_name` | `net0` | The standardized name to apply to the primary network interface |

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
        network_interface_rename__new_interface_name: "eth0"
```

## How It Works

1. Checks if the target interface name already exists on the system
2. If the interface doesn't exist:
   - Identifies the current primary interface and its MAC address
   - Finds the existing netplan configuration file
   - Modifies the netplan configuration to use the standardized interface name
   - Applies the new configuration and waits for network to stabilize
3. Displays detailed information about the network interface after renaming

## Verification

After the role executes successfully, you can verify the implementation with:

```bash
# Check interface name
ip a

# Verify netplan configuration
ls -la /etc/netplan/
cat /etc/netplan/*.yaml

# Check network connectivity
ping -c 4 8.8.8.8
```

The role also outputs detailed information about the renamed interface at the end of execution, including:
- IP address
- Gateway
- Interface name
- MAC address
- Netmask
- Network
- Prefix

## Testing

To test this role in isolation before deploying to production:

1. Run the role with `--check` mode first to preview changes
2. Apply the role to a test system with a simple network configuration
3. Verify network connectivity is maintained after interface renaming
4. Reboot the system to verify persistence of the interface name

## Limitations and Considerations

- This role is designed specifically for systems using Netplan as the network configuration tool
- The role only supports a **single** netplan configuration file
- Running this role may cause momentary network interruptions during the interface renaming process
- If the target interface name already exists, the role will skip the renaming process