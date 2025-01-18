# Configure systemd-remote-upload via Ansible

This role provides a way to configure a central logging server via
systemd-remote-upload. Unlike other roles, **this role specifically focuses on
the client-side**.

It is assumed that some kind of server-side software already exists. Options
include, but are not limited to:

- systemd-journald-remote
- VictoriaLogs

The role is published to [Ansible Galaxy](https://galaxy.ansible.com/ui/namespaces/drkhsh)

## Parameters

See full list at [defaults.yml](./defaults/main.yml)

| Parameter                                 | Description                        | Default                                          |
|-------------------------------------------|------------------------------------|--------------------------------------------------|
| journald_upload_server                    | Server to send the logs to         | `loghost.example.com`                            |
| journald_upload_server_ssl                | Use SSL to send the logs           | `false`                                          |
| journald_upload_server_ssl_ca_certificate | CA certificate of the server       | `{{ journald_ssl_ca_certificates }}`             |
| journald_upload_client_retention          | Retention time of journald         | `0`                                              |
| journald_upload_client_ssl_cert           | Client certificate                 | `-`                                              |
| journald_upload_client_ssl_key            | Client key                         | `-`                                              |
| journald_upload_client_storage            | Journal storage                    | `auto`                                           |

## Example playbook

This example assumes that a Let's Encrypt certificate and no client
authentication is used, the host's local log storage will be *volatile* with a
retention of *1 hour* and the log destination is a *VictoriaLogs* server.

```
- name: "Setup systemd-journal-upload remote logging"
  hosts: all
  tags:
    - journald
  vars:
    journald_server: 'loghost.example.org:9428/insert/journald'
    journald_server_ssl: true
    journald_client_storage: 'volatile'
    journald_client_retention: '1h'
  roles:
    - drkhsh.journald_upload
```

### SSL client authentication

When running this role with SSL client authentication enabled, make sure you
have a CA certificate ready and are able to create client certificates for the
systemd-journal-upload user.

A sample script to install the certificates:

```
- name: 'Ensure journald client certificate group'
  group:
    name: 'journald-cert'
    system: true
    state: 'present'

- name: 'Ensure journald client certificate directory'
  file:
    path: '/etc/ssl/journald'
    state: 'directory'
    owner: 'root'
    group: 'journald-cert'
    mode: 0770

- name: 'Ensure certificate files'
  copy:
    src: "{{ cert.src }}"
    dest: "{{ cert.dest }}"
    owner: 'root'
    group: 'journald-cert'
    mode: 0440
  loop:
    - src: "{{ journald_certificate_source }}"
      dest: '/etc/ssl/journald/client.crt'
    - src: "{{ journald_key_source }}"
      dest: '/etc/ssl/journald/client.key'
  loop_control:
    loop_var: 'cert'
```
