# ansible-productivity-apps

Ansible playbooks that make productivity apps match a declared configuration. Each playbook loads one private
declaration document at run time and passes its sections to the roles of a public collection. The first app is
JetBrains YouTrack, configured through the roles of the `jpe_ansible.youtrack` collection.

## Requirements

- Ansible 2.17 or later.
- On the controller, the Python libraries `hvac` and `boto3`.
- Access to the object store that holds the declaration document, and to the credential store that holds the
  object-store credentials and the Hub credential of the admin user.

## Install the collections

```sh
ansible-galaxy collection install -r requirements.yml
```

`requirements.yml` pins the `jpe_ansible.youtrack` collection, the `dryvist.homelab` collection that provides the
`inventory_resolve` role, and the `amazon.aws` and `community.hashi_vault` collections.

## Declaration document

The playbook reads one JSON or YAML document. Its top-level key is `youtrack`. Each section under `youtrack` uses the
variable names of one role. The file [`examples/youtrack.example.json`](examples/youtrack.example.json) shows the
shape. Its values are invented.

| Key | Role | Holds |
| --- | --- | --- |
| `youtrack.youtrack_url` | all | Base URL of the YouTrack server. |
| `youtrack.bundles` | `jpe_ansible.youtrack.bundles` | The six `bundles_*` lists. Only `bundles_enum` and `bundles_state` are required. |
| `youtrack.custom_fields` | `jpe_ansible.youtrack.custom_fields` | `custom_fields_declared`. |
| `youtrack.link_types` | `jpe_ansible.youtrack.link_types` | `link_types_declared`. |

The role README files list the fields of each entry.

## Environment variables

The converge playbook, `site.yml`, stops before it contacts a server when a required variable is missing. The repo holds
no value for any of these. It holds only the names. The export playbook's variables are listed in its own section.

| Variable | Required | Purpose |
| --- | --- | --- |
| `TOFU_INVENTORY_S3_URI` | One of the two document variables | URI of the object that holds the declaration document. |
| `TOFU_INVENTORY_PATH` | One of the two document variables | Local path of a copy of the declaration document. |
| `BAO_ADDR` | Yes | Address of the credential store. |
| `BAO_TOKEN` | Yes | Token that may read the object-store credentials and the Hub credential of the admin user. |
| `YOUTRACK_ADMIN_OPENBAO_MOUNT` | Yes | KV version 2 mount that holds the Hub credential of the admin user. |
| `YOUTRACK_ADMIN_OPENBAO_PATH` | Yes | Secret path that holds the Hub credential. |
| `YOUTRACK_ADMIN_OPENBAO_FIELD` | Yes | Field of that secret that holds the permanent token of the admin user. |
| `YOUTRACK_ADMIN_LOGIN` | Yes | Login of the admin user. The play stops unless exactly one Hub user has this login. |
| `YOUTRACK_HUB_URL` | No | Hub REST base URL. Defaults to `<youtrack_url>/hub/api/rest`, with `youtrack_url` from the document. |
| `YOUTRACK_HUB_SERVICE_ID` | No | Hub id of the YouTrack service. Set it to skip the lookup by application name. |

## Run

Run a check first.

```sh
ansible-playbook -i localhost, site.yml --check
```

A `--check` run of the converge playbook creates one temporary permanent token of the admin user and deletes it before
the run ends. It makes no other write: every role runs in check mode and only reports the changes it would make.

Run again without `--check` to apply the declaration. Each run creates one temporary token and deletes it, also when a
role fails. Apart from that token, a run that finds nothing to change writes nothing.

The play mints and deletes the token with the task files `playbooks/tasks/temp_token_mint.yml` and
`playbooks/tasks/temp_token_delete.yml`, which the export playbook includes too. The Hub REST base URL is
`<youtrack_url>/hub/api/rest`, with `youtrack_url` from the document, unless `YOUTRACK_HUB_URL` is set.

## Usage

### Export the server configuration

`playbooks/youtrack_export.yml` writes the configuration of a YouTrack server to one local JSON file. It runs the read-only
`jpe_ansible.youtrack.export` role with a temporary permanent token of the admin user. The play:

1. Reads the Hub credential of the admin user from the credential store.
2. Finds the admin user by login and the YouTrack service in the Hub services list, unless `YOUTRACK_HUB_SERVICE_ID` is set.
3. Creates one permanent token for the admin user, scoped to the YouTrack service.
4. Runs the export role with that token and writes the file.
5. Deletes the token by its id, also when the export fails.

The playbook makes exactly two writes, the create and the delete of its own temporary token. Every other request is a
read. In check mode the play stops before it creates the token, because an export cannot be simulated.

| Variable | Required | Purpose |
| --- | --- | --- |
| `YOUTRACK_URL` | Yes | Base URL of the server, for example `https://youtrack.example.com`. |
| `YOUTRACK_HUB_URL` | No | Hub REST base URL. Defaults to `<YOUTRACK_URL>/hub/api/rest`, the built-in Hub of the server. |
| `BAO_ADDR` | Yes | Address of the credential store. |
| `BAO_TOKEN` | Yes | Token that may read the Hub credential. |
| `YOUTRACK_ADMIN_OPENBAO_MOUNT` | Yes | KV version 2 mount that holds the Hub credential. |
| `YOUTRACK_ADMIN_OPENBAO_PATH` | Yes | Secret path that holds the Hub credential. |
| `YOUTRACK_ADMIN_OPENBAO_FIELD` | Yes | Field of that secret that holds the permanent token of the admin user. |
| `YOUTRACK_ADMIN_LOGIN` | Yes | Login of the admin user. The play stops unless exactly one Hub user has this login. |
| `YOUTRACK_HUB_SERVICE_ID` | No | Hub id of the YouTrack service. Set it to skip the lookup by application name. |
| `YOUTRACK_EXPORT_DEST` | Yes | Path of the JSON file to write. |

```sh
ansible-playbook playbooks/youtrack_export.yml
```

The playbook is not part of `site.yml`. Run it on demand.

## Contributing

Open a pull request against `main`. Keep each commit to one change. Use generic examples only:
`https://youtrack.example.com` and the project short name `DEMO`.

## License

Apache License 2.0. See [LICENSE](LICENSE).
