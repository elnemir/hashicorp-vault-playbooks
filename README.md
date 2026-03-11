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

## Качество
- `ansible-lint`
- `yamllint`
- `pre-commit`
