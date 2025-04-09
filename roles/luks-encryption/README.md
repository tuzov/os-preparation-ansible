# Ansible Role: luks-encryption

## Description

This Ansible role implements LUKS (Linux Unified Key Setup) encryption for disks and partitions on Linux systems. It supports encrypting both entire secondary disks and individual partitions on the primary disk, creating secure encrypted volumes with automated key management. The role handles the complete encryption lifecycle including key generation, LUKS setup, filesystem creation, and mounting.

The role uses cryptsetup to implement encryption, generates secure keyfiles for automated unlocking, and configures the system for persistent mounting through /etc/crypttab entries.

## ⚠️ CRITICAL WARNING ⚠️
NEVER encrypt the BIOS boot partition on the primary disk! This partition (typically 4MB in size and marked with bios_grub flag) is essential for system boot. Encrypting this partition will render your system completely unbootable and may require full reinstallation.
Before proceeding with encryption, always identify the BIOS boot partition:
```
# lsblk
xvda     202:0    0   18G  0 disk
├─xvda1  202:1    0   17G  0 part /
├─xvda14 202:14   0    4M  0 part
├─xvda15 202:15   0  106M  0 part /boot/efi
└─xvda16 259:0    0  913M  0 part /boot

# parted /dev/xvda print
Model: Xen Virtual Block Device (xvd)
Disk /dev/xvda: 19.3GB
Sector size (logical/physical): 512B/512B
Partition Table: gpt
Disk Flags:

Number  Start   End     Size    File system  Name  Flags
14      1049kB  5243kB  4194kB                     bios_grub
```

As shown above, partition 14 has the bios_grub flag and is 4MB - this partition must never be encrypted.
Also avoid encrypting:

* EFI System Partition (often mounted at /boot/efi)
* /boot partition
* Active root partition without proper initramfs configuration

## Requirements

- Debian/Ubuntu-based Linux distribution
- Ansible 2.9+
- Root access on target hosts
- One of the following:
  - A secondary disk to encrypt completely
  - A partition on the primary disk to encrypt

## Role Variables

| Variable | Default | Description |
|----------|---------|-------------|
| `local_secret_dir` | `{{ playbook_dir }}/secrets` | Local directory for keyfiles and headers |
| `remote_secret_dir` | `/dev/shm` | Remote in-memory directory for sensitive operations |
| `keyfiles_dir` | `cryptsetup-keys` | Subdirectory for keyfiles |
| `headers_dir` | `luks-headers` | Subdirectory for LUKS headers |
| `keyfile_size` | `256` | Size of encryption key in bits |
| `keyfile_source_dev` | `/dev/random` | Source for random keyfile generation |
| `keyfile_cipher_default` | `aes-cbc-essiv:sha256` | Default cipher for encryption |
| `keyfile_cipher_light` | `aes-xts-plain64` | Lightweight cipher for small partitions |
| `manage_filesystem` | `true` | Create filesystem on encrypted device |
| `manage_mount` | `true` | Mount encrypted device after setup |
| `fstype` | `ext4` | Filesystem type to create |
| `disk_mount` | `/data` | Mount point for encrypted disk |
| `partition_mount` | `/tiny-data` | Mount point for encrypted partition |
| `mount_owner` | `root` | Owner of mount point directory |
| `mount_group` | `root` | Group of mount point directory |
| `shred_before_encryption` | `false` | Shred partitions before encryption (time-consuming) |
| `header_backup` | `true` | Create LUKS header backups |

## Dependencies

None.

## Example Playbook

```yaml
---
- hosts: servers
  become: true
  roles:
    - role: luks-encryption
      tags: luks
```

```yaml
#inventory.ini
[os_preparation]

[os_preparation:vars]
secondary_disk=/dev/xvdf
partition_on_primary=/dev/xvda14
```

With custom configuration:

```yaml
#inventory.ini
[os_preparation]

[os_preparation:vars]
secondary_disk=/dev/sdb
partition_on_primary=/dev/sda14
disk_mount=/encrypted-data
partition_mount=/encrypted-home
mount_owner=appuser
mount_group=appgroup
```

## How It Works

1. Validates required variables and checks if target devices exist and are not mounted
2. Installs required packages (cryptsetup, coreutils)
3. For disk encryption:
   - Creates GPT partition table if needed
   - Generates secure keyfiles for encryption
   - Creates LUKS container on the device
   - Adds entries to /etc/crypttab for persistent configuration
   - Creates filesystem and mounts the encrypted volume
   - Creates LUKS header backups for recovery
4. For partition encryption:
   - Follows similar steps as disk encryption without partition table manipulation

## Verification

After the role executes successfully, you can verify the implementation with:

```bash
# Check if devices appear in crypttab
cat /etc/crypttab

# Verify the encrypted devices are available
ls -la /dev/mapper/encrypted*

# Confirm mount points are accessible
df -h | grep encrypted

# Validate that data can be written to encrypted volumes
touch /data/test-file
```

## Testing

To test this role in isolation before deploying to production:

1. Run the role with `--check` mode first to preview changes
2. Apply the role to a test system with non-critical data
3. Reboot the system to verify persistent mounting
4. Test recovery procedures using the header backups

## Limitations and Considerations

- This role may overwrite existing data on target unmounted disks/partitions
- The encryption process cannot be easily reversed without potential data loss
- For critical data, consider implementing additional backup strategies
- The keyfiles are generated automatically and should be securely stored
- Mounts do not survive reboot as keyfile is located on `/dev/shm`