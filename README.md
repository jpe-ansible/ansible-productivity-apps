# ansible-productivity-apps

Ansible playbooks that make productivity apps match a declared configuration. Each playbook loads one private
declaration document at run time and passes its sections to the roles of a public collection. The first app is
JetBrains YouTrack, configured through the roles of the `jpe_ansible.youtrack` collection.

## Requirements

- Ansible 2.17 or later.
- On the controller, the Python libraries `hvac` and `boto3`.
- Access to the object store that holds the declaration document, and to the credential store that holds the
  object-store credentials and the YouTrack API key.

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
| `youtrack.bundles` | `jpe_ansible.youtrack.bundles` | `bundles_enum` and `bundles_state`. |
| `youtrack.custom_fields` | `jpe_ansible.youtrack.custom_fields` | `custom_fields_declared`. |
| `youtrack.link_types` | `jpe_ansible.youtrack.link_types` | `link_types_declared`. |

The role README files list the fields of each entry.

## Environment variables

The playbook stops before it contacts a server when a required variable is missing. The repo holds no value for any
of these. It holds only the names.

| Variable | Required | Purpose |
| --- | --- | --- |
| `TOFU_INVENTORY_S3_URI` | One of the two document variables | URI of the object that holds the declaration document. |
| `TOFU_INVENTORY_PATH` | One of the two document variables | Local path of a copy of the declaration document. |
| `BAO_ADDR` | Yes | Address of the credential store. |
| `BAO_TOKEN` | Yes | Token that may read the object-store credentials and the YouTrack API key. |
| `YOUTRACK_OPENBAO_MOUNT` | Yes | KV version 2 mount that holds the YouTrack API key. |
| `YOUTRACK_OPENBAO_PATH` | Yes | Secret path that holds the YouTrack API key. |
| `YOUTRACK_OPENBAO_FIELD` | Yes | Field of that secret that holds the API key. |

## Run

Run a check first. It reads the server and writes nothing.

```sh
ansible-playbook -i localhost, site.yml --check
```

Run again without `--check` to apply the declaration. A run that finds nothing to change writes nothing.

## Contributing

Open a pull request against `main`. Keep each commit to one change. Use generic examples only:
`https://youtrack.example.com` and the project short name `DEMO`.

## License

Apache License 2.0. See [LICENSE](LICENSE).
