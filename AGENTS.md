# AI Agents Configuration

Ansible playbook repo. It loads one declaration document at run time and passes its sections to the roles of the
public `jpe_ansible.youtrack` collection. It declares no values of its own.

## Rules

- Do not add `.py`, `.sh`, or `.rb` files. Use Ansible modules, plugins and Jinja.
- Use generic examples only: `https://youtrack.example.com`, the project short name `DEMO`, and invented names.
- Never write a real host, address, bucket, secret path, token or declared value into this repo. Read each location
  from an environment variable, and name only the variable in the repo.
- Every task that handles a credential sets `no_log: true`.
- Keep each role's variable names. Pass document sections to roles without renaming or translating them.
- Do not add a second fetch of the document. The `dryvist.homelab.inventory_resolve` role loads it.
