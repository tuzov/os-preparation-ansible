# OS Preparation Ansible Playbook

This Ansible playbook automates server OS preparation tasks for Linux systems, implementing disk encryption, performance optimization, and standardization of network interface names across infrastructure.

## Features

- **LUKS Encryption**: Secures data at rest through full disk and/or partition encryption
- **CPU Performance Optimization**: Maximizes processing speed by optimizing CPU states and governors
- **Network Interface Standardization**: Ensures consistent interface naming across servers
- **CPU Information Display**: Provides detailed CPU information for system verification

## Requirements

- Ansible 2.9+
- Ubuntu 
- Required Ansible collections:
  ```
  ansible-galaxy collection install community.general
  ansible-galaxy collection install community.crypto
  ansible-galaxy collection install ansible.posix
  ```

## Directory Structure

```
.
├── README.md
├── inventory.ini
├── os-preparation.yaml
├── copy-luks-keys.yaml
└── roles
    ├── cpu-performance-optimisation/
    ├── luks-encryption/
    └── network-interface-rename/
```

## Configuration

Edit your `inventory.ini` file to specify target servers and required configuration:

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

Encrypts disks and partitions using LUKS, with automated key management and mounting. 
For more details, see [luks-encryption README](roles/luks-encryption/README.md).

To copy keyfiles and headers to remote machines after reboot, use the included `copy-luks-keys.yaml` helper playbook.

⚠️ Warning!
Never encrypt the BIOS boot partition (commonly 4MB with bios_grub flag). Verify partitions with parted /dev/sda print before encryption.

### CPU Performance Optimization

Optimizes CPU settings for high performance by disabling CPU idle states and setting performance governors.
For more details, see [cpu-performance-optimisation README](roles/cpu-performance-optimisation/README.md).

### Network Interface Rename

Standardizes network interface naming to ensure consistency across server fleet.
For more details, see [network-interface-rename README](roles/network-interface-rename/README.md).