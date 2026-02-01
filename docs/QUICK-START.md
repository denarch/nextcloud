# Что делать сейчас — чеклист для новичка

Проект залит на GitHub: https://github.com/denarch/nextcloud

---

## Вы уже сделали

- [x] Добавили APP_SECRET в `.env`
- [x] Создали репозиторий на GitHub
- [x] Проект подключён к GitHub

---

## Что нужно сделать вам (по порядку)

### 1. Скачайте проект на Synology NAS

1. Откройте **File Station** на Synology
2. Перейдите в папку `docker` (создайте, если нет)
3. Создайте папку `filestash-r2`
4. На компьютере откройте: https://github.com/denarch/nextcloud
5. Нажмите **Code** → **Download ZIP**
6. Распакуйте архив
7. Загрузите в `docker/filestash-r2/` файлы:
   - `docker-compose.yml`
   - `.env.example`
8. Переименуйте `.env.example` в `.env`
9. Откройте `.env` и убедитесь, что там ваш APP_SECRET (вы уже его добавили)

### 2. Установите и запустите Filestash на Synology

Следуйте инструкции: [SYNOLOGY-SETUP.md](SYNOLOGY-SETUP.md)

Кратко:
1. Установите **Container Manager** из Package Center
2. Container Manager → **Project** → **Create** → **Create from docker-compose**
3. Укажите путь `/volume1/docker/filestash-r2`
4. Запустите проект
5. Откройте в браузере: `http://IP-вашего-NAS:8334`
6. Задайте пароль администратора
7. Подключите R2: Admin → Storage → S3 (см. README)

### 3. Настройте Cloudflare Tunnel

Следуйте инструкции: [CLOUDFLARE-TUNNEL.md](CLOUDFLARE-TUNNEL.md)

Кратко:
1. Зайдите на https://one.dash.cloudflare.com
2. **Networks** → **Tunnels** → **Create a tunnel** → Cloudflared
3. Скопируйте токен
4. Добавьте Public Hostname: `files` → `denarchvisuals.com` → `http://host.docker.internal:8334`
5. В Container Manager создайте контейнер из образа `cloudflare/cloudflared`
6. В команде запуска укажите: `tunnel run --token ВАШ_ТОКЕН`

### 4. Готово

Откройте https://files.denarchvisuals.com

---

## Если что-то не получается

- Filestash не открывается → проверьте логи в Container Manager → Project → filestash-r2 → Logs
- Туннель не работает → используйте IP NAS вместо `host.docker.internal` (например `http://192.168.1.100:8334`) в настройках Public Hostname
- R2 не подключается → проверьте endpoint `https://<ACCOUNT_ID>.r2.cloudflarestorage.com` и ключи из Cloudflare Dashboard
