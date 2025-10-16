
# Ansible Multi-Cloud VMware Migration

**Flow:** Discover (vCenter) → Landing Zone (per cloud) → Migration (cold/live) → Cutover

## Quickstart
```bash
ansible-galaxy install -r requirements.yml
pipx inject ansible pyvmomi boto3 botocore google-auth google-api-python-client azure-cli

# Put secrets in Vault (vCenter, cloud creds)
ansible-vault create group_vars/vault.yml

# Edit group_vars/* for CIDRs, buckets, names, regions
# Discover VMware
ansible-playbook -i inventories/vmware.yml plays/00_discover_vmware.yml -t discover --ask-vault-pass

# Build landing zones
ansible-playbook site.yml --tags 'aws,landing'   --ask-vault-pass
ansible-playbook site.yml --tags 'azure,landing' --ask-vault-pass
ansible-playbook site.yml --tags 'gcp,landing'   --ask-vault-pass

# Cold image migration
ansible-playbook site.yml --tags 'aws,cold'   --ask-vault-pass
ansible-playbook site.yml --tags 'azure,cold' --ask-vault-pass
ansible-playbook site.yml --tags 'gcp,cold'   --ask-vault-pass

# Live migration (agents/services)
ansible-playbook site.yml --tags 'aws,live'   --ask-vault-pass
ansible-playbook site.yml --tags 'azure,live' --ask-vault-pass
ansible-playbook site.yml --tags 'gcp,live'   --ask-vault-pass

# Cutover
ansible-playbook site.yml --tags cutover --ask-vault-pass
```

### Notes
- Secrets go in `group_vars/vault.yml` (encrypted). A plaintext **sample** is provided as `group_vars/vault.sample.yml`.
- Image import steps are not strictly idempotent across clouds. For production, consider wrapping `import-image` operations in CI or provider-native workflows.
- Expand networking (VPN/Direct Connect/ExpressRoute/Interconnect) as needed; current roles create basic VPC/VNet with subnets and a simple IGW/firewall where applicable.
