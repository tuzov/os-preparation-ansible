# Ansible Role: cpu-performance-optimisation

## Description

This Ansible role optimizes CPU performance on servers by disabling CPU C-states, setting tuned profile for latency-performance, and configuring the CPU frequency governor to performance mode. These optimizations reduce latency and improve processing speed at the cost of increased power consumption, making this role ideal for high-performance computing environments, database servers, and low-latency applications.

The role uses a combination of sysfs modifications, tuned profiles, and a custom systemd service to ensure CPU optimizations persist across reboots, even in environments like AWS where kernel parameter modifications are restricted.

## Requirements

- Linux-based target systems (Ubuntu/Debian compatible)
- Ansible 2.9+
- Root access on target hosts
- systemd

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `cpu_performance_optimisation__configure_tuned` | `true` | Whether to install and configure the tuned service |
| `cpu_performance_optimisation__tuned_profile` | `latency-performance` | The tuned profile to use for performance optimization |
| `cpu_performance_optimisation__disable_cstates_dir` | `/opt/scripts` | Directory where C-state disabling script will be placed |

## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: all
  become: true
  roles:
    - role: cpu-performance-optimisation
      tags: cpu
```

With custom tuned profile:

```yaml
---
- hosts: all
  become: true
  roles:
    - role: cpu-performance-optimisation
      tags: cpu
      vars:
        cpu_performance_optimisation__tuned_profile: "powersave"
```

## How It Works

1. CPU C-states Optimization:
   - Identifies CPU cores in the system
   - Creates a systemd service that disables CPU idle states on boot
   - Deploys a script that writes to sysfs to force C-states to disabled state

2. CPU Governor Configuration:
   - Installs and configures the tuned service for CPU performance management
   - Sets the CPU frequency scaling governor to "performance" mode
   - Applies performance-optimized tuned profile

## Verification

After the role executes successfully, you can verify the implementation with:

```bash
# Check if C-states are disabled
cat /sys/devices/system/cpu/cpu*/cpuidle/state*/disable

# Verify CPU governor settings
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor

# Check active tuned profile
tuned-adm active

# Check all the available profile
tuned-adm profile
```

The role also outputs information about CPU governor configuration at the end of execution.

## Testing

To test this role in isolation before deploying to production:

1. Run the role with `--check` mode first to preview changes
2. Apply the role to a non-production system
3. Use performance benchmarking tools to verify improvement
4. Verify changes persist after system reboot

## Limitations and Considerations

- AWS doesn't allow disabling of C-states through kernel parameters in GRUB. As a workaround, this role uses a systemd service method.
- Virtual machines may have limited access to certain CPU performance features
- Not all systems support the same CPU frequency governors
- Increased power consumption is expected as a trade-off for improved performance