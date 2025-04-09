# does not work on aws, thus disabled by default
disable_cstates_persistent: false

# Disable C-state persistently if machine allows
- name: Persistent C-state disabling
  - name: Check if C-states disabling parameters exist in GRUB command line
    lineinfile:
      path: /etc/default/grub
      regexp: '^GRUB_CMDLINE_LINUX=".*{{ disable_cstates_grub_cmdline }}.*"$'
      state: absent
    check_mode: true
    register: grub_cstates_check
    changed_when: false

  - name: Add C-states disabling parameters to GRUB command line if missing
    lineinfile:
      backrefs: true
      path: /etc/default/grub
      regexp: "^(GRUB_CMDLINE_LINUX=\".*)\"$"
      line: "\\1 {{ disable_cstates_grub_cmdline }}\""
    when: grub_cstates_check.found == 0
    notify: update grub
  when: disable_cstates_persistent

as a workaround tuned profile is used
