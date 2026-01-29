# AAP-CaC (AAP 2.4/AWX ➜ AAP 2.5/2.6) — Migration Starter Pack

> **Important support notice!!**  
> This repository is a **community starting point** for moving Automation Controller objects that are exportable via **Config-as-Code (CaC)** from **AAP 2.4 or AWX** into **AAP 2.5+**.  
> It is **not** an official or supported migration path from Red Hat. Validate everything in non-production first.

> **Credential secrets cannot be migrated**  
> CaC **does not export secret values**. During import, **credential objects are created without their passwords/tokens/private keys**. You must **re-enter secrets manually** in AAP 2.5 (or seed them via your own secret manager/API) after the import completes.

## What this repo contains

- **`export_old.yml`** — exports CaC data from **AAP ≤ 2.4 / AWX**. Use this when your source is 2.4 or older.  
- **`export_new.yml`** — exports CaC data from **AAP 2.5+** (useful for iterative backups or 2.5→2.5 transfers).  
- **`import_new.yml`** — imports the exported CaC bundle into **AAP 2.5** in dependency-safe order.  
- **`vars.yml`** — example variables file you customize for your environments.  
- **`collections/`** — pinned collections for repeatable runs (install with `ansible-galaxy`).  

---

## Prerequisites

- **Ansible** on your runner host.  
- Network/API access to **source controller** (AAP 2.4/AWX) and **destination controller** (AAP 2.5).  
- **Personal Access Tokens** (PATs) with admin-level permissions for both.  
- (Recommended) a non-prod target for dry runs.

Install required collections (from the included lock/requirements under `collections/` if present):

```bash
ansible-galaxy install -r collections/requirements.yml
```

---

## Configure `vars.yml`

Create or edit `vars.yml` in the repo root. The exact variable names in your playbooks may vary slightly; here’s a **working template**:

```yaml
# AAP 2.4 or AWX
controller_username: "admin" # MUST BE ADMIN USER
controller_password: "<your password here>"
controller_hostname: "controller.example.com" # DO NOT INCLUDE http:// or https://
controller_api_plugin: awx.awx.controller_api

## 2.5/2.6
aap_username: "admin" # MUST BE ADMIN USER
aap_password: "<your password here>"
aap_hostname: "aap.example.com" # DO NOT INCLUDE http:// or https://

# General
export_organization: "{{ default(None) }}"
output_path: "./exports/{{ export_organization }}"
flatten_output: true
aap_validate_certs: "false"
controller_validate_certs: "false"

# DEBUG
controller_configuration_credentials_secure_logging: "false"
cas_secure_logging: "false"
```

> Keep tokens out of Git. Prefer environment variables + `ansible-vault` or your secret manager.

---

## Run order (copy/paste)

### A) Export from AAP 2.4 / AWX
```bash
ansible-playbook export_old.yml -e @vars.yml -e export_organization=<Org Name>
```

### B) Export from AAP 2.5+
```bash
ansible-playbook export_new.yml -e @vars.yml
```

### C) Import into AAP 2.5+
```bash
ansible-playbook import_new.yml -e @vars.yml
```

> **After the import:** re-enter all credential secrets in AAP 2.5.

---

## Typical object coverage

- Organizations, teams, users
- Credential types, **credentials (metadata only)**  
- Projects, inventories/groups/hosts, inventory sources  
- Execution environments, notification templates  
- Job templates, workflow job templates, schedules, RBAC mappings

---

## Tips & troubleshooting

- **401/403**: validate tokens and `*_validate_certs`.  
- **Project sync fails**: re-enter SCM credentials.  
- **Credential test fails**: re-enter secrets.  
- **Re-runs**: imports are idempotent.

---

## FAQ

**Is this an official Red Hat migration?**  
No.

**Why aren’t passwords exported?**  
Secret values are encrypted at rest and not emitted by CaC.

---

