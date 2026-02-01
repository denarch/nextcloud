# Cloudflare Tunnel — доступ к Filestash через files.denarchvisuals.com

Cloudflare Tunnel позволяет открыть Filestash в интернет без проброса портов роутера. SSL настраивается автоматически.

---

## Что вам понадобится

1. Synology NAS с установленным Filestash (см. [SYNOLOGY-SETUP.md](SYNOLOGY-SETUP.md))
2. Домен denarchvisuals.com, добавленный в Cloudflare
3. Cloudflare Zero Trust (бесплатный план)

---

## Шаг 1: Включите Cloudflare Zero Trust

1. Перейдите на https://one.dash.cloudflare.com
2. Войдите в аккаунт Cloudflare
3. Выберите вашу команду (team) или создайте новую
4. Введите имя (например: Personal)

---

## Шаг 2: Создайте туннель

1. В меню слева: **Networks** → **Tunnels**
2. Нажмите **Create a tunnel**
3. Выберите **Cloudflared**
4. Введите имя туннеля: `filestash-synology`
5. Нажмите **Save tunnel**

---

## Шаг 3: Скопируйте токен

После создания туннеля Cloudflare покажет команду для запуска, например:

```
docker run cloudflare/cloudflared:latest tunnel --no-autoupdate run --token XXXXX...
```

**Скопируйте весь текст после `--token`** (длинная строка) — это ваш Tunnel Token. Он понадобится на следующем шаге.

---

## Шаг 4: Настройте Public Hostname

1. В интерфейсе туннеля найдите **Public Hostname**
2. Нажмите **Add a public hostname**
3. Укажите:
   - **Subdomain:** `files` (получится files.denarchvisuals.com)
   - **Domain:** выберите `denarchvisuals.com`
   - **Service type:** HTTP
   - **URL:** `http://host.docker.internal:8334`
4. Нажмите **Save hostname**

> **Если `host.docker.internal` не работает:** замените на IP вашего NAS в локальной сети, например `http://192.168.1.100:8334`

---

## Шаг 5: Установите cloudflared в Docker на Synology

1. Откройте **Container Manager** на Synology
2. Перейдите в **Registry**
3. Найдите `cloudflare/cloudflared`
4. Скачайте образ (Download)

---

## Шаг 6: Создайте контейнер cloudflared

1. В Container Manager: **Container** → **Create**
2. Выберите образ **cloudflare/cloudflared**
3. Нажмите **Advanced settings**

**General:**
- Имя: `cloudflared-filestash`
- Включите **Enable auto-restart**

**Network:**
- Выберите **Use the same network as Docker Host** (если есть)
- Или оставьте bridge — тогда в Шаге 4 укажите IP NAS вместо host.docker.internal

**Execute command:**
Вместо стандартной команды введите:

```
tunnel run --token ВСТАВЬТЕ_ВАШ_ТОКЕН_СЮДА
```

Замените `ВСТАВЬТЕ_ВАШ_ТОКЕН_СЮДА` на токен из Шага 3.

4. Нажмите **Apply** → **Next** → **Done**
5. Запустите контейнер

---

## Шаг 7: Проверьте

1. Откройте https://files.denarchvisuals.com
2. Должен открыться Filestash

---

## Возможные проблемы

**502 Bad Gateway / страница не открывается**

1. Убедитесь, что Filestash запущен и доступен по `http://IP-NAS:8334` в браузере с той же сети
2. Если использовали `host.docker.internal` — попробуйте IP NAS (например `http://192.168.1.100:8334`) в настройках Public Hostname
3. Пересоздайте Public Hostname с правильным URL

**Туннель показывает "Disconnected"**

1. Проверьте, что контейнер cloudflared запущен
2. Проверьте правильность токена
3. В логах контейнера (Container → cloudflared-filestash → Details → Logs) посмотрите ошибки
