# Ansible Role: Gitleaks for RHEL 9 and Derivatives

[[_TOC_]]

## Description

This Ansible role installs Gitleaks, a fast and secure Git secrets scanner, either in offline or online mode, based on the configuration. The role ensures that the specified version of Gitleaks is installed, and optionally verifies the installation via checksum validation when using the offline mode.

It supports installation on Red Hat Enterprise Linux 9 and its derivatives, including AlmaLinux 9, Rocky Linux 9, and Oracle Linux 9.

## Requirements

- Ansible 2.12 or higher
- Access to the internet (for online installation) or a pre-downloaded Gitleaks binary (for offline installation)
- Root privileges or `become: true` access is required, as the role installs Gitleaks into `/usr/local/bin` and performs system-level operations
- One of the following operating systems:
  - Red Hat Enterprise Linux 9
  - AlmaLinux 9
  - Rocky Linux 9
  - Oracle Linux 9

## Dependencies

None.

## Tasks Overview

1. **Check if Gitleaks is already installed:** Verifies whether the desired version of Gitleaks is already installed on the system.

2. **Install Gitleaks (Offline Mode):** Installs the Gitleaks binary from a pre-existing file (provided by the user) if the system is in offline mode.

3. **Install Gitleaks (Online Mode):** Downloads and installs Gitleaks from the official GitHub releases in online mode.

4. **Verify Gitleaks Installation:** Verifies that the installed Gitleaks binary matches the desired version.

5. **Clean Up Temporary Files (Online Mode):** Cleans up any temporary files created during the installation process in online mode.

## Role Variables

The role has the following [default](./defaults/main.yml) variables that must be verified as per the requirements before using the role:

Variable                  | Default Value | Description
------------------------- | ------------- | ------------------------------------------------------------------------------------------------------------------
`configure_offline`       | `false`       | If `true`, uses a pre-provided Gitleaks binary [gitleaks](./files/gitleaks) for version 8.25.0. You may also provide your own binary and set the expected checksum using `offline_gitleaks_sha256`. If `false`, downloads Gitleaks from GitHub.
`gitleaks_version`        | `"8.25.0"` | Version of Gitleaks to install if `configure_offline` is set to `false`.
`gitleaks_dest`           | `"/usr/local/bin/gitleaks"` | Final installation path for the Gitleaks binary.
`gitleaks_download_url`   | `"https://github.com/gitleaks/gitleaks/releases/download/v{{ gitleaks_version }}/gitleaks_{{ gitleaks_version }}_linux_x64.tar.gz"` | URL to download Gitleaks from GitHub (used in online mode).
`gitleaks_tmp_dir`        | `"{{ ansible_env.HOME }}/.gitleaks_tmp_install"` | Temporary directory used during online installation.
`offline_gitleaks_sha256` | `"a4b6ec120e0d4da50370e7ae64d0ef17f2d120d0f6143931a3061e3992f00565"` | SHA256 checksum to verify the Gitleaks binary during offline installation.

## Role Usage

This ansible role supports both offline and online installation modes, depending on whether or not the target system has internet access. To customize behavior, set the appropriate variables as shown below.

### 1. Offline Mode

In offline mode, you must provide the Gitleaks binary file and its SHA256 checksum for verification. Use the `configure_offline` variable and set it to true.

Example usage in a playbook:

```yaml
- name: Install Gitleaks in offline mode
  hosts: all
  become: true
  roles:
    - role: gitleaks_EL
      vars:
        configure_offline: true
        gitleaks_dest: "/usr/local/bin/gitleaks"
        offline_gitleaks_sha256: "a4b6ec120e0d4da50370e7ae64d0ef17f2d120d0f6143931a3061e3992f00565" # Checksum changes if you change the binary in the files directory
```
Ensure the Gitleaks binary file is available in the files/ directory.

### 2. Online Mode

Example usage in a playbook:

```yaml
- name: Install Gitleaks in online mode
  hosts: all
  become: true
  roles:
    - role: gitleaks_EL
      vars:
        configure_offline: false
        gitleaks_version: "8.25.0"
```

## Testing

This role includes both Molecule tests and smoke tests.

### Molecule Testing

This role comes with Molecule testing for both offline and online modes. The test suite includes:
- Syntax checking
- Role application
- Idempotence test
- Verification that gitleaks is installed and working

#### Running Tests (Offline Mode)

To test the role with offline mode using Molecule, make sure the gitleaks binary is placed in the `files/` directory.

Run the following command to test:

```bash
molecule test -s offline
```

#### Running Tests (Online Mode)

To test the role with online mode using Molecule, run the following command:

```bash
molecule test -s online
```

### Smoke Testing

A simple smoke test is available in the `tests` directory. To run it on the localhost (defined by default in the inventory), use the following commands:

```bash
cd tests
ansible-playbook test.yml -K
```
The `-K` flag tells Ansible to prompt you for the sudo password.

If you want to target specific hosts, you should pass an inventory with a `-i` flag as follows:

```bash
cd tests
ansible-playbook -i inventory test.yml -K
```

The smoke test verifies:
- Role application
- Gitleaks installation
- Gitleaks version check

⚠️ **Important Note About Smoke Tests** ⚠️

When running smoke tests on your local machine, such as using localhost as the target host, the role may make changes to your local system. These changes include:
- Installing or updating Gitleaks
- Modifying permissions on /usr/local/bin/gitleaks
- Creating directories or copying files

If you are testing the role on your local machine (e.g., using localhost), please be aware that it will apply these changes directly to your system.

## License

Apache-2.0

## Author Information

This role was created by the NDAAL Team, Pierre Gronau, Ayesha Shafqat.
