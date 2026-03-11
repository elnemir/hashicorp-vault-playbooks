# Changelog

## 2026-03-11

### Added
- Создан каталог `docs/`.
- Добавлен `docs/Project.md` с целевой архитектурой Vault RAFT на RedOS 8 в VMware 7.0.3.
- Добавлен `docs/Tasktracker.md` с эпиками, задачами, статусами и приоритетами.
- Добавлен `docs/Diary.md` с первой записью наблюдений/решений/проблем.
- Добавлен `docs/qa.md` с полным списком уточняющих вопросов для финализации плейбука.

### Updated
- `docs/Tasktracker.md`: добавлена критическая задача по сбору ответов на `qa.md` (статус `В процессе`).
- `docs/Diary.md`: добавлена запись о старте этапа Discovery и текущих блокерах.
- `docs/Project.md`: добавлены подтвержденные параметры среды (узлы, сеть, TLS, bootstrap, DevOps-практики) и перечень блокеров.
- `docs/Tasktracker.md`: обновлены статусы discovery-задач, добавлены новые задачи по governance/operations.
- `docs/qa.md`: добавлены разделы `Закрыто` и `Остались блокеры` по результатам ответов заказчика.
- `docs/Diary.md`: добавлена запись о полученных ответах и незакрытых рисках.
- `docs/Project.md`: зафиксированы `Ansible 2.12+`, `prometheus`, `RPO/RTO`, snapshot-параметры и maintenance window.
- `docs/Tasktracker.md`: обновлен прогресс по задачам operations/delivery, оставлены только актуальные блокеры.
- `docs/qa.md`: сокращен список блокеров до governance и приемки.
- `docs/Diary.md`: добавлена запись о частичном закрытии Discovery-блокеров.
- `docs/Project.md`: зафиксированы `key_custodians Option C`, `root_token_policy Option A`, `mandatory_tests=functional+failover`.
- `docs/qa.md`: обновлен статус ответов, блокеры сокращены до `key_custodians roles` и `owner_team`.
- `docs/Tasktracker.md`: задача по процессу поставки переведена в `Завершена`, governance-задача в `В процессе`.
- `docs/Diary.md`: добавлена запись о выборе governance-модели.
- `docs/Project.md`: зафиксированы `owner_team=IT-Team` и роли `key_custodians`.
- `docs/qa.md`: блокеры закрыты.
- `docs/Tasktracker.md`: задачи Discovery переведены в `Завершена`, этап готов к реализации плейбука.
- `docs/Diary.md`: добавлена запись о полном закрытии Discovery-блокеров.
- `docs/Tasktracker.md`: добавлена и завершена задача создания каркаса репозитория; задачи ролей переведены в `В процессе`.
- `docs/Diary.md`: добавлены записи о старте реализации и создании каркаса Ansible.
- `docs/Tasktracker.md`: добавлен прогресс по каждой роли на этапе скелета реализации.

### Added
- Добавлен `ansible.cfg`.
- Добавлены конфигурации качества: `.ansible-lint`, `.yamllint`, `.pre-commit-config.yaml`.
- Добавлены inventories: `inventories/dev`, `inventories/stage`, `inventories/prod` с базовыми `hosts.yml` и `group_vars`.
- Добавлены playbooks: `playbooks/site.yml`, `playbooks/os-baseline.yml`, `playbooks/vault-cluster.yml`.
- Добавлены роли:
  - `roles/os_baseline` (`defaults/tasks/handlers`)
  - `roles/vault_install` (`defaults/tasks/handlers/templates`)
  - `roles/vault_config` (`defaults/tasks/handlers/templates`)
  - `roles/vault_bootstrap` (`defaults/tasks`)
- Обновлен `README.md` с требованиями, структурой и командами запуска.
- Обновлен `README.md` с ключевыми production-переменными (`checksum`, strict firewall, bootstrap/audit).
- Добавлен `docs/runbook.md` с эксплуатационными процедурами для IT-Team.

### Refined
- В стартовых ролях синхронизированы defaults с inventory-переменными (`NTP`, `SELinux`, `firewalld`, `vault_api_addr`).
- Улучшена обработка статусов `firewall-cmd` в `roles/vault_config/tasks/main.yml` для более корректной идемпотентности.
- Исправлен порядок запуска сервиса Vault: старт перенесен на этап `vault_config`, чтобы исключить запуск до генерации `vault.hcl`.
- Усилен `roles/os_baseline`: добавлено управление SSH-firewall правилами по CIDR и strict-режим удаления общего SSH-правила.
- Усилен `roles/vault_install`: добавлен checksum-aware download и удаление временного архива Vault.
- Усилен `roles/vault_config`: проверки TLS-артефактов, выставление прав доступа, CIDR-ограничения для `8200/8201`, strict/non-strict firewall режимы.
- Усилен `roles/vault_bootstrap`: безопасные проверки status/init/unseal, поддержка включения audit-device с root token.
- Обновлены `inventories/prod/group_vars/*` для strict firewall-политики и bootstrap-параметров audit.
- `docs/Tasktracker.md`: hardening-итерация 1 переведена в `Завершена`.
- `docs/Diary.md`: добавлена запись по hardening-итерации 1.
- `docs/Tasktracker.md`: добавлена задача `Production-ready доработка ролей (итерация 2)` и активирован runbook-трек.
- `docs/Diary.md`: добавлена запись о старте итерации 2.
- `docs/Tasktracker.md`: runbook-задача переведена в `Завершена`, обновлен прогресс итерации 2.
- `docs/Diary.md`: добавлена запись с результатами итерации 2.
- Улучшен `roles/vault_config`: добавлен post-check `vault status -format=json` после запуска сервиса.
- Улучшен `roles/vault_install`: добавлен контроль версии установленного бинарника Vault.
- Улучшен `roles/vault_bootstrap`: реализована логика `effective_root_token` и `token revoke -self` в соответствии с policy `Option A`.
- Исправлены firewalld rich-rules (`os_baseline`, `vault_config`) на безопасную передачу через `argv`.
