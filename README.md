# Nginx static test server

В репозитории представлен Dockerfile, docker-compose.yaml и playbook Ansible для настройки сервера раздачи статики. Роли описывают базовую настройку сервера, настраивают nginx, который должен раздавать изображения по пути `/images` отсюда [https://tinyurl.com/5dzjhdn6](https://tinyurl.com/5dzjhdn6).

## Тестовое окружение

В качестве тестового окружения, к которому будет подключаться Ansible, используется Docker-контейнер, запускаемый через Docker Compose. Для этого контейнера используется Dockerfile, размещённый в docker директории,  используется в качестве базового образа официальный Ubuntu 24.04. Вместе с ОС установливается `openssh-server` для доступа по SSH. Также описана конфигурация Docker Compose с пробросом необходимых портов.

## Реализация
В целях оптимизации времени и скорости настройки в данной работе приняты некоторые решения, которые можно исправить для использования в продуктовой среде.
1. Развернуть тестовое окружение из директории docker через `docker-compose up -d --build --force-recreate`.
2. Запустить плейбук ANSIBLE_HOST_KEY_CHECKING=False ansible-playbook -i inventory.yml playbook.yml -u ansible_user --ask-pass --ask-become-pass Необходимая информация укзаана в Dockerfile.  Плейбук запускается с сервисным пользователем заданным в Dockerfile. 
3. На 80 порту, по пути `/images` отображается структура файлов, а по пути `/images/filename` отображается изображение.
```
root@f20de0519fa1:~# curl -I http://127.0.0.1:80/images/cloud1.jpg
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Thu, 30 Jan 2025 12:20:17 GMT
Content-Type: image/jpeg
Content-Length: 763887
Last-Modified: Wed, 29 Jan 2025 12:32:00 GMT
Connection: keep-alive
ETag: "679a1fc0-ba7ef"
Expires: Thu, 30 Jan 2025 13:20:17 GMT
Cache-Control: max-age=3600
Cache-Control: public, max-age=3600
Accept-Ranges: bytes
```
5. SSH порт открыт для подключения.

## Реализованы следующие роли для Ansible плейбука

1. **Роль для создания/удаления пользователей/групп пользователей** - role: users
   - Состояние пользователей и список кастомных групп необходимо указан в общем group_vars/all.yml файле.
   - Устанавливаются пользователям следующие параметры:
     - Имя
     - Оболочка (bash/zsh/etc)
     - Состояние (удален/создан)
     - Публичный ключ (для подключения по SSH)
     - Пароль
     - Список групп, в которых состоит пользователь (в том числе кастомные группы, которые задаются в `all.yml`).

2. **Роль для установки zsh/ohmyzsh для пользователей**: role: zsh
   - Устанавливается только для пользователей с соответственным shell.

3. **Роль для настройки SSH**:
   - Запрещён вход с пользователя `root`.
   - Запрещены пустые пароли.
   - Выставлено логгирование в режим `VERBOSE`.
   - Отключен `X11Forwarding`.

4. **Обновление пакетов и установка дополнительных утилит**: role: utils
   - Список легко расширяем, так как указан в group_vars/all.yml в секции packages:
     - `htop`
     - `ncdu`
     - `git`
     - `nano`
     - `unzip`
     - `bzip2`
     - `curl`

5. **Роль для раскатки nginx**: role: nginx
   - Nginx устанавливается, настраивается, создается `vhost` с нужным `locations`.
   - Из настроек сделано:
     - Настроено gzip-ирование для статики.
     - Логирование запросов.
     - Добавлено кэширование с сроком истечения 1 час.

6. **Роль для заливки статики**: role: static
   - Роль выкачивает по static_url из group_vars/all.yml архив clouds.zip со статикой размещённой в этом репозитории в Releases
