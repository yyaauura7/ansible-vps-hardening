# Автоматизация настройки и защиты VPS (Ansible Roles)

Модульная конфигурация **Infrastructure as Code (IaC)** на базе **Ansible** для первичной настройки и усиления безопасности сервера на **Ubuntu 24.04 LTS**. Проект переработан из монолитного плейбука в переиспользуемые роли по промышленным стандартам (`defaults/`, `tasks/`, `handlers/`, `templates/`).

## Архитектура и состав ролей

```text
[ Control Node (macOS) ] ──(SSH + ed25519)──▶ [ Target VPS (Ubuntu 24.04) ]
                                                     │
playbook.yml ──▶ 1. role: ssh       (юзер devops, вход только по ключу, запрет root/паролей)
             ──▶ 2. role: ufw       (политика deny incoming, открытие портов 22, 80, 443)
             ──▶ 3. role: fail2ban  (шаблон jail.local.j2 с backend = systemd)
             ──▶ 4. role: nginx     (установка и автозапуск веб-сервера)
```

## Структура репозитория

```text
.
├── .gitignore                 # Исключает inventory.ini с реальным IP и .DS_Store из Git
├── ansible.cfg                # Настройки подключения, путь к inventory и отключение retry-файлов
├── inventory.ini.example      # Безопасный шаблон инвентаря для репозитория
├── playbook.yml               # Главный сценарий запуска модульных ролей
├── roles/
│   ├── ssh/                   # Настройка пользователя sudo, SSH-ключей и защита sshd_config
│   │   ├── defaults/main.yml
│   │   ├── handlers/main.yml
│   │   └── tasks/main.yml
│   ├── ufw/                   # Управление правилами файрвола UFW через переменные
│   │   ├── defaults/main.yml
│   │   └── tasks/main.yml
│   ├── fail2ban/              # Защита от брутфорса с конфигурацией на базе шаблона Jinja2
│   │   ├── defaults/main.yml
│   │   ├── handlers/main.yml
│   │   ├── tasks/main.yml
│   │   └── templates/jail.local.j2
│   └── nginx/                 # Установка и управление службой Nginx
│       ├── defaults/main.yml
│       └── tasks/main.yml
└── README.md                  # Документация проекта
```

## Инженерные стандарты и безопасность

- **Модульность и переиспользование:** Каждая подсистема изолирована в отдельной роли со своими значениями по умолчанию (`defaults/main.yml`), что позволяет подключать роли выборочно на разные типы серверов.
- **Шаблонизация конфигураций (Jinja2):** Параметры Fail2Ban вынесены в шаблон `roles/fail2ban/templates/jail.local.j2` с подстановкой переменных времени блокировки и числа попыток.
- **Безопасная правка SSH (`validate`):** Изменения `/etc/ssh/sshd_config` проверяются командой `/usr/sbin/sshd -t -f %s` до перезапуска демона, исключая потерю доступа к серверу.
- **Идемпотентность и обработчики (`handlers`):** Службы `ssh` и `fail2ban` перезапускаются через механизм `notify` только при реальном изменении конфигурационных файлов.

## Проверка и запуск

1. Скопируйте шаблон инвентаря и укажите IP-адрес вашего сервера:
   ```bash
   cp inventory.ini.example inventory.ini
   ```
2. Проверьте синтаксис плейбука и всех подключённых ролей:
   ```bash
   ansible-playbook --syntax-check playbook.yml
   ```
3. Выполните пробный прогон без внесения изменений на сервере (`dry-run`):
   ```bash
   ansible-playbook --check --diff playbook.yml
   ```
4. Примените конфигурацию на целевом сервере:
   ```bash
   ansible-playbook playbook.yml
   ```
