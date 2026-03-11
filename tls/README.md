# TLS input directory

Поместите TLS-файл(ы) для доставки на Vault-ноды.

## Вариант 1: Единый файл (bundle mode)
- Путь по умолчанию: `tls/vault.pem`
- Используется при:
  - `vault_tls_deploy_from_controller: true`
  - `vault_tls_controller_mode: bundle`

## Вариант 2: Раздельные файлы (split mode)
- `tls/vault.crt`
- `tls/vault.key`
- `tls/ca.crt`
- Используется при:
  - `vault_tls_deploy_from_controller: true`
  - `vault_tls_controller_mode: split`

Секретные файлы из `tls/` исключены из Git через `.gitignore`.
