# OS Preparation Ansible Playbook

This Ansible playbook automates server preparation tasks for Linux systems, implementing security hardening, performance optimization, and standardization across infrastructure.

## Features

- **LUKS Encryption**: Secures data at rest through full disk and/or partition encryption
- **CPU Performance Optimization**: Maximizes processing speed by optimizing CPU states and governors
- **Network Interface Standardization**: Ensures consistent interface naming across servers
- **Display of CPU Information**: Displays CPU information at the playbook end

## Requirements

- Ansible 2.9+
- Ubuntu 18.04+ or other Debian-based distributions
- Target servers with SSH access and sudo privileges
- Required Ansible collections:
  ```
  ansible-galaxy collection install community.general
  ansible-galaxy collection install community.crypto
  ```

## Directory Structure

```
.
├── README.md
├── inventory.ini
├── os-preparation.yaml
└── roles
    ├── cpu-performance-optimisation
    ├── luks-encryption
    └── network-interface-rename
```

## Configuration

Edit `inventory.ini` to specify your target servers and required configuration:

```ini
[os_preparation]
server1.example.com
server2.example.com

[os_preparation:vars]
# LUKS Configuration
secondary_disk=/dev/sdb       # Disk to encrypt (optional)
partition_on_primary=/dev/sda14  # Partition to encrypt (optional)
```

## Usage

### Basic Execution

To run the complete playbook:

```bash
ansible-playbook -i inventory.ini os-preparation.yaml
```

### Selective Role Execution

To run specific roles using tags:

```bash
# Run only disk encryption tasks
ansible-playbook -i inventory.ini os-preparation.yaml --tags luks

# Run only CPU optimization tasks
ansible-playbook -i inventory.ini os-preparation.yaml --tags cpu

# Run only network interface tasks
ansible-playbook -i inventory.ini os-preparation.yaml --tags interface
```

## Role Details

### LUKS Encryption

Encrypts disks and partitions using LUKS, with automated key management and persistent mounting. See [luks-encryption README](roles/luks-encryption/README.md) for details.

#### ⚠️ Warning!
Never encrypt the BIOS boot partition (commonly 4MB with `bios_grub` flag). Verify partitions with `parted /dev/sda print` before encryption. See [luks-encryption README warning](roles/luks-encryption/README.md#⚠️-critical-warning-⚠️) for more detailed information.

### CPU Performance Optimization

Optimizes CPU settings for high performance by disabling CPU idle states and setting performance governors. See [cpu-performance-optimisation README](roles/cpu-performance-optimisation/README.md) for details.

### Network Interface Rename

Standardizes network interface naming to ensure consistency across server fleet. See [network-interface-rename README](roles/network-interface-rename/README.md) for details.
