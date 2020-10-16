### Раскатка ansible роли users через gitlab CI/CD

Что делает роль: создает пользователей ОС, в authorized_key каждого прописывая публичный ключ некого человека (оператора/программиста/админа). Если у пользователя ОС предполагается несколько публичных ключей в authorized_key, то роль users необходимо будет править.

Некоторые моменты:
- для успешного job нужен runner с тегом `users`
- gitlab runner был выбран c executor _docker_, базовый конфиг:

```
cat /etc/gitlab-runner/config.toml
concurrent = 1
check_interval = 0

[session_server]
  session_timeout = 1800

[[runners]]
  name = "test_runner"
  url = "https://gitlab...."
  token = "...."
  executor = "docker"
  [runners.docker]
    tls_verify = false
    image = "williamyeh/ansible:centos7"
    privileged = false
    disable_entrypoint_overwrite = false
    oom_kill_disable = false
    disable_cache = false
    volumes = ["/cache", "/var/run/docker.sock:/var/run/docker.sock", "/home/gitlab/.ssh:/root/.ssh/"]
    shm_size = 0
```

В папке `/home/gitlab/.ssh` раннера лежит приватный ключ, с помощью которого подключаемся под пользователем _ansible_ к управляемым через anisble хостам (соответственно, данный юзер имеет sudo права для выполнения команд).

- чтобы ansible начал читать конфиг файл _ansible.cfg_ в репозитории с ролью, пришлось в шаге _before_script_ выставлять более верные права (репо имеет имя `test_users` и находится в gitlab группе `test`).  
