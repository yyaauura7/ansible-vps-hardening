# Ansible VPS Hardening (Ubuntu 24.04)

Idempotent Ansible playbook for automated initial provisioning and security hardening of an Ubuntu 24.04 VPS.

## Overview / О проекте
Этот проект упаковывает базовую настройку и защиту чистого облачного сервера (Ubuntu 24.04) в воспроизводимый код (**Infrastructure as Code**). Плейбук работает без установки агентов на сервер (agentless) поверх SSH и гарантирует идемпотентность: повторный запуск проверяет конфигурацию и не вносит лишних изменений.

## Key Features / Что настраивает плейбук
- **User & Access:** Создаёт непривилегированного пользователя `devops` с правами `sudo` и добавляет публичный SSH-ключ.
- **SSH Hardening:** Отключает прямой вход под `root` и аутентификацию по паролю в `/etc/ssh/sshd_config` с предварительной валидацией синтаксиса (`sshd -t`).
- **Firewall (UFW):** Включает политику запрета всех входящих соединений по умолчанию (`default deny incoming`) и открывает только порты `22` (SSH), `80` (HTTP) и `443` (HTTPS).
- **Intrusion Prevention (Fail2Ban):** Устанавливает Fail2Ban и настраивает защиту `sshd` с параметром `backend = systemd` для Ubuntu 24.04.
- **Web Server (Nginx):** Устанавливает веб-сервер Nginx и добавляет его в автозагрузку системы.

## Project Structure / Структура проекта
- `ansible.cfg` — локальные настройки Ansible (инвентарь по умолчанию, тихий режим Python).
- `inventory.ini.example` — безопасный шаблон адресной книги серверов (реальный `inventory.ini` скрыт в `.gitignore`).
- `playbook.yml` — основной идемпотентный сценарий настройки и защиты сервера.
- `README.md` — документация проекта.

## Quick Start / Быстрый старт
1. Подготовьте файл инвентаря (укажите IP вашего сервера, пользователя и путь к приватному SSH-ключу):
   `cp inventory.ini.example inventory.ini`
2. Запустите безопасную симуляцию (Dry Run), чтобы проверить изменения без правки файлов на сервере:
   `ansible-playbook -K --check --diff playbook.yml`
3. Примените конфигурацию на сервере:
   `ansible-playbook -K playbook.yml`
