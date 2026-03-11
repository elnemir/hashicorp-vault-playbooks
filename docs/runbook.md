# Runbook: HashiCorp Vault RAFT on RedOS 8

## 1. Назначение
Документ описывает эксплуатационные процедуры для кластера Vault (`dev/stage/prod`) с параметрами проекта:
- 3-node RAFT cluster
- HAProxy VIP: `10.255.107.49`
- unseal model: Shamir (`5 shares`, `threshold 3`)
- owner team: `IT-Team`

## 2. Ответственность
- Эксплуатационный владелец: `IT-Team`
- Key custodians: `IT Director`, `Dev Lead`, `Net Admin`, `Win Admin`, `Lin Admin`

## 3. Предусловия перед деплоем
1. TLS-материалы доступны одним из способов:
   - Ручная доставка на ноды (`/etc/vault.d/tls/vault.crt`, `/etc/vault.d/tls/vault.key`, `/etc/vault.d/tls/ca.crt`)
   - Controller-side доставка из каталога проекта (`tls/`) через переменные `vault_tls_deploy_from_controller` и `vault_tls_controller_mode`.
2. Проверена сетевой доступ:
   - `8200/tcp` только от разрешенных API CIDR
   - `8201/tcp` только между нодами кластера
3. Установлен Ansible `2.12+` на control-host.
4. Проверены inventory/variables в `inventories/prod/group_vars/`.

## 3.1 Единый URL для браузера
- Используется переменная `vault_public_url` (например `https://10.255.107.49:8200` или DNS-имя LB).
- UI и API должны открываться через этот URL.

Проверка:

```bash
VAULT_PUBLIC_URL="https://10.255.107.49:8200"
curl -k "${VAULT_PUBLIC_URL}/v1/sys/health"
```

## 4. Базовые команды
Синтаксическая проверка:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml --syntax-check
```

Полный запуск:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/site.yml
```

Если используете единый TLS-файл из репозитория:
1. Положите файл в `tls/vault.pem`.
2. Убедитесь, что в inventory включено:
   - `vault_tls_deploy_from_controller: true`
   - `vault_tls_controller_mode: bundle`

Только baseline ОС:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/os-baseline.yml
```

Только Vault кластер:

```bash
ansible-playbook -i inventories/prod/hosts.yml playbooks/vault-cluster.yml
```

## 5. Первичный bootstrap (Shamir/manual)
По умолчанию bootstrap-role выключен (`vault_bootstrap_enabled=false`), что безопасно для production.

### 5.1 Инициализация вручную
На выбранном лидере (или через `VAULT_ADDR` на VIP):

```bash
export VAULT_ADDR=https://10.255.107.49:8200
export VAULT_CACERT=/etc/vault.d/tls/ca.crt
vault operator init -key-shares=5 -key-threshold=3
```

Сохранить unseal keys и initial root token строго по утвержденной governance-политике.

### 5.2 Manual unseal
На каждой sealed-ноде:

```bash
vault operator unseal <key1>
vault operator unseal <key2>
vault operator unseal <key3>
```

Проверка:

```bash
vault status
vault operator raft list-peers
```

## 6. Audit device (day-1)
Целевой формат: file audit в `/var/log/vault/audit.log`.

Ручное включение (если не включено):

```bash
vault audit enable -path=file file file_path=/var/log/vault/audit.log
```

Проверка:

```bash
vault audit list
```

## 7. Root token policy
Согласованная политика: bootstrap-only (`Option A`) с немедленным revoke после завершения day-1 операций.

```bash
vault token revoke -self
```

Примечание: длительное хранение root token запрещено.

## 8. Failover проверка (mandatory)
Минимальный набор приемки включает:
- `functional`
- `failover`

Шаги failover:
1. Определить текущего лидера:

```bash
vault status
```

2. Остановить Vault на лидере:

```bash
systemctl stop vault
```

3. Убедиться в перевыборах и доступности API через VIP.
4. Запустить лидера обратно:

```bash
systemctl start vault
```

## 9. Snapshot и восстановление
Параметры проекта:
- Schedule: `daily 03:00`
- Storage: `nfs`
- RPO: `1 hour`
- RTO: `24 hours`

### 9.1 Создание snapshot вручную
```bash
vault operator raft snapshot save /nfs/vault/snapshots/vault-$(date +%F-%H%M).snap
```

### 9.2 Восстановление
1. Перевести кластер в maintenance.
2. Выбрать целевой snapshot.
3. Выполнить:

```bash
vault operator raft snapshot restore /nfs/vault/snapshots/<snapshot>.snap
```

4. Проверить статус нод, peers и доступность API.

## 10. Обновление Vault (rolling)
1. Обновить `vault_version` (и checksum) в `inventories/prod/group_vars/vault.yml`.
2. Применять по одной ноде (`serial: 1` уже задан в `playbooks/vault-cluster.yml`).
3. После каждой ноды проверять:
   - `vault status`
   - `vault operator raft list-peers`
   - API доступ через VIP.

## 11. Troubleshooting
- Проверка systemd:

```bash
systemctl status vault
journalctl -u vault -n 200 --no-pager
```

- Проверка TLS-файлов:
  - права `vault:vault`
  - ключ `0600`, сертификаты `0640`

- Проверка firewall:

```bash
firewall-cmd --list-rich-rules
firewall-cmd --list-ports
```

## 12. Операционный чек-лист после каждого изменения
1. `vault status` на всех нодах.
2. `vault operator raft list-peers`.
3. Проверка API через VIP.
4. Проверка audit-device (`vault audit list`).
5. Проверка актуальности snapshot по расписанию.
