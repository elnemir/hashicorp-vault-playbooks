# hashicorp-vault-playbooks

Ansible-плейбуки для развертывания кластера HashiCorp Vault (RAFT) на RedOS 8 в VMware 7.0.3.

## Требования
- Ansible `2.12+`
- SSH-доступ к целевым хостам
- TLS-материалы Vault, размещенные на узлах в `/etc/vault.d/tls/`

## Структура
- `inventories/` - окружения `dev`, `stage`, `prod`
- `playbooks/` - точка входа и сценарии запуска
- `roles/` - роли `os_baseline`, `vault_install`, `vault_config`, `vault_bootstrap`
- `docs/` - архитектура, трекер задач, дневник и журнал изменений

## Быстрый запуск
Проверка синтаксиса:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml --syntax-check
```

Применение к `prod`:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

## Важные переменные
- `vault_download_checksum` - SHA256 checksum архива Vault (рекомендуется заполнить для production).
- `vault_config_firewall_strict` - строгие firewall-правила для Vault портов (CIDR-based).
- `os_baseline_strict_ssh_firewall` - удаляет общее правило `ssh` и оставляет только разрешенные CIDR.
- `vault_bootstrap_enabled` и `vault_bootstrap_auto_init` - управление bootstrap-флоу.
- `vault_bootstrap_root_token` - требуется для автоматического включения audit-device.
- `vault_public_url` - единый URL для доступа к UI/API через LB.
- `vault_tls_deploy_from_controller` - включить доставку TLS из каталога проекта.
- `vault_tls_controller_mode` - `bundle` (единый файл) или `split` (3 файла).

## Качество
- `ansible-lint`
- `yamllint`
- `pre-commit`

## Эксплуатация
- Runbook: [docs/runbook.md](/Users/eln/Documents/hashicorp-vault-playbooks/docs/runbook.md)
- TLS input: [tls/README.md](/Users/eln/Documents/hashicorp-vault-playbooks/tls/README.md)
