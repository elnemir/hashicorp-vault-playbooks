# QA: Уточняющие вопросы для финализации архитектуры и плейбука

Ниже список вопросов, ответы на которые позволят собрать production-ready плейбук с минимальным числом проходов.

## Статус ответов (2026-03-11)

### Закрыто
- Топология: 3 узла, HAProxy VIP `10.255.107.49`, anti-affinity `true`.
- Узлы: `vault-test-01/02/03` с параметрами 4 vCPU / 8 GB RAM / 50 GB disk.
- Окружения: `dev`, `stage`, `prod`.
- Базовая модель ОС: full update, NTP `10.277.107.1`, SELinux `disabled`, firewalld управляется Ansible.
- Сетевая модель: API и SSH из `10.0.0.0/8`, порт `8201` только межузловой, API только через LB.
- TLS: `internal-ca`, `pem`, ручная доставка сертификатов, ротация 365 дней, mTLS клиентам не требуется.
- Bootstrap: `shamir_manual`, `key_shares=5`, `key_threshold=3`, audit device day-1 включен.
- Day-1 функционал: `AppRole` + `KV v2`, OSS-режим.
- Логирование: `syslog`.
- DevOps-практики: `ansible-vault`, pre-commit, CI checks, check mode, rolling update.
- Версия Ansible: `2.12+`.
- Monitoring: `prometheus`.
- Целевые значения: `RPO=1 hour`, `RTO=24 hours`.
- Snapshot: `daily 03:00`, storage: `nfs`.
- Maintenance window: `Sun 03:00-06:00 MSK`.
- Модель `key_custodians`: `Option C` (5 доверенных лиц + офлайн escrow у ИБ).
- `unseal_key_storage`: `trusted persons`.
- Роли `key_custodians`: `IT Director`, `Dev Lead`, `Net Admin`, `Win Admin`, `Lin Admin`.
- `root_token_policy`: `Option A` (bootstrap-only и немедленный revoke).
- `mandatory_tests`: `базовый` (`functional`, `failover`).
- `owner_team`: `IT-Team`.

### Остались блокеры
- На текущем этапе блокеры отсутствуют.

## 1. Топология и инфраструктура VMware
1. Сколько узлов Vault планируется: 3 или 5?
2. Какие ресурсы у каждой ВМ (vCPU, RAM, disk, тип datastore)?
3. Есть ли требования к anti-affinity rules в VMware для узлов Vault?
4. Используется ли LB/VIP перед Vault API? Если да, какой именно (NSX, HAProxy, F5 и т.д.)?
5. Какие FQDN и IP уже выделены для каждого узла и для (возможного) VIP?
6. Есть ли отдельные среды (dev/stage/prod), и требуется ли единый плейбук под все среды?

## 2. RedOS 8 и базовая ОС-конфигурация
1. Какие внутренние репозитории доступны для обновлений (URL/доступ)?
2. Требуется ли полный `dnf update` до последнего доступного уровня?
3. Какие обязательные baseline-пакеты должны быть установлены кроме Vault?
4. Нужны ли корпоративные настройки proxy, DNS search domains, NTP/chrony?
5. Есть ли требования по SELinux (enforcing/permissive + кастомные policy)?
6. Какие требования к journald/logrotate/auditd?

## 3. Сетевая безопасность
1. Подтвердите список разрешенных портов:
   - Vault API (`8200/tcp`)
   - Vault cluster (`8201/tcp`)
   - SSH/Ansible (`22/tcp`)
2. Между какими сетевыми зонами должен быть разрешен доступ к API?
3. Нужно ли ограничение доступа к API только через LB/VIP?
4. Требуется ли настройка firewalld в рамках плейбука или это зона ответственности сетевой команды?

## 4. TLS/PKI
1. Кто выпускает сертификаты: внутренний CA, AD CS, Vault PKI или внешний CA?
2. В каком формате будут передаваться сертификаты/ключи (`pem`, `pfx`), и где хранить их до деплоя?
3. Какие SAN должны быть в сертификатах (FQDN узла, IP, VIP)?
4. Нужна ли взаимная TLS-аутентификация (mTLS) для клиентов?
5. Какой регламент ротации сертификатов и срок действия?

## 5. Vault bootstrap и security model
1. Какая модель unseal требуется:
   - Shamir + ручной unseal,
   - auto-unseal (HSM/KMS)?
2. Сколько `key_shares` и `key_threshold` нужно для init?
3. Кто и где хранит unseal keys и initial root token?
4. Нужен ли запрет на хранение чувствительных bootstrap-данных в логах Ansible?
5. Нужно ли сразу после init выполнять hardening действий (отключение root token, создание admin policy/token, audit device)?

## 6. Политики и интеграции Vault
1. Какие auth methods нужно включить на старте (userpass/LDAP/Kubernetes/JWT/AppRole/OIDC)?
2. Какие secret engines нужны в первом релизе (KV v2, PKI, Transit, Database и др.)?
3. Нужна ли интеграция с LDAP/AD и какие параметры подключения?
4. Нужна ли namespace-модель (если Enterprise) или OSS-режим?

## 7. Эксплуатация, наблюдаемость и DR
1. Куда отправлять логи Vault (локально, syslog, ELK/Splunk)?
2. Какие health-check и monitoring инструменты используются (Prometheus, Zabbix, Datadog и др.)?
3. Какой целевой RPO/RTO?
4. Как часто делать RAFT snapshot и где хранить бэкапы?
5. Есть ли отдельные требования к процедуре восстановления из snapshot?

## 8. Ansible-процесс и DevOps
1. Какая версия Ansible должна использоваться в проекте?
2. Нужны ли pre-commit hooks и CI-проверки в этом репозитории?
3. Где будут храниться `ansible-vault` пароли/ключи и кто имеет доступ?
4. Требуется ли поддержка `check mode` и `diff mode` для всех ролей?
5. Нужна ли стратегия поэтапного деплоя (rolling update) и maintenance windows?

## 9. Критерии приемки
1. Какие тесты обязательны перед приемкой (functional, failover, security scan)?
2. Какая команда будет финальным владельцем эксплуатации?
3. Нужны ли формальные артефакты: отчет по проверкам, runbook, схема доступа?

---

После ваших ответов:
- Обновлю `Project.md` с финальными архитектурными решениями.
- Переведу задачи в `Tasktracker.md` в статус исполнения.
- Сформирую структуру Ansible-репозитория и начну реализацию ролей.
