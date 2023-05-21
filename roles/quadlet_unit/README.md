`samh.podman_systemd.quadlet_unit`
==================================

This role installs and configures a systemd service unit for podman
container(s) using [Quadlet](https://www.redhat.com/sysadmin/quadlet-podman).
Quadlet was added in Podman 4.4.

Full documentation can be found in the
[podman-systemd.unit](https://docs.podman.io/en/latest/markdown/podman-systemd.unit.5.html)
man page.

Requirements
------------

Podman should already be installed and configured to run rootless on the target
system (probably we should add another role for this).

Role Variables
--------------

The following variables are required:

- `quadlet_unit_name` - the name of the systemd service unit, not including
  the extension
- `quadlet_unit_user` - the user to run the systemd service unit (currently
  only rootless podman is supported)

The following variables are optional:

- `quadlet_unit_extension` - the type of quadlet unit; can be any supported
  type including `container`, `volume`, `network`, and `kube`
  (default: `container`)
- `quadlet_unit_description` - the description of the systemd service unit
  (only used if the `Unit` section is not specified in `quadlet_unit_config`)
- `quadlet_environment` - a dictionary of environment variables to set for the
  systemd service unit
- `quadlet_unit_state` - the desired state of the systemd service unit
  (default: `started`); can be any state supported by the
  `ansible.builtin.systemd` module, or `null` to leave the unit state alone

The unit file can be generated in multiple ways:

- `quadlet_unit_config` - a dictionary of unit file configuration options
- `quadlet_unit_template` - a path to a unit file template to use instead
  of generating one from the built-in template

Dependencies
------------

A list of other roles hosted on Galaxy should go here, plus any details in regards to parameters that may need to be set for other roles, or variables that are used from other roles.

Example Playbook
----------------
Generating a `container` unit using `quadlet_unit_config`:

```yaml
- hosts: servers
  vars:
    # These are some example variables (not directly related to the role)
    # that might be defined in, for example, `group_vars`, inventory, or the
    # defaults file of another role.
    podgrab_user: podgrab
    podgrab_image_tag: latest
    podgrab_port: 8080

  tasks:
    - name: configure systemd unit
      ansible.builtin.import_role:
        name: samh.podman_systemd.quadlet_unit
      vars:
        quadlet_unit_name: podgrab
        quadlet_unit_user: "{{ podgrab_user }}"
        quadlet_unit_config:
          Unit:
            Description: Podgrab podcast manager
            Wants: network-online.target
            After: network-online.target
          Container:
            Image: "docker.io/akhilrex/podgrab:{{ podgrab_image_tag }}"
            PublishPort: "{{ podgrab_port }}:8080"
            # For options that are listed multiple times, use a list
            # of strings (*not a list of dictionaries*)
            Environment:
              - "CHECK_FREQUENCY=240"
              - "PASSWORD=supersecret"
          Install:
            # Start by default on boot
            WantedBy: multi-user.target default.target

```

Generating a `container` unit using a template (if you want more control over
the unit file or just prefer to use systemd syntax directly instead of
translating it to a YAML dictionary):

```yaml
- hosts: servers
  tasks:
    - name: configure systemd unit
      ansible.builtin.import_role:
        name: samh.podman_systemd.quadlet_unit
      vars:
        quadlet_unit_name: podgrab
        quadlet_unit_user: podgrab
        quadlet_unit_template: podgrab.container.j2
```
