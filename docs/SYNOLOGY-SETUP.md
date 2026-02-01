# Установка Filestash на Synology NAS

Пошаговая инструкция для новичков. Выполняйте шаги по порядку.

---

## Шаг 1: Установите Container Manager (Docker)

1. Откройте **Package Center** (Центр пакетов) на вашем Synology
2. Найдите **Container Manager**
3. Нажмите **Install** (Установить)
4. Дождитесь завершения установки

---

## Шаг 2: Создайте папку для проекта

1. Откройте **File Station**
2. Перейдите в `docker` (если нет — создайте в корне тома)
3. Создайте папку `filestash-r2`
4. Путь должен быть: `/volume1/docker/filestash-r2/`

---

## Шаг 3: Скопируйте файлы на NAS

**Вариант A: через File Station (если вы скачали проект с GitHub)**

1. Скачайте проект с GitHub: https://github.com/denarch/nextcloud
2. Нажмите **Code** → **Download ZIP**
3. Распакуйте архив на компьютере
4. В File Station перейдите в `docker/filestash-r2/`
5. Загрузите туда файлы:
   - `docker-compose.yml`
   - `.env.example` (скопируйте и переименуйте в `.env`)
   - В `.env` замените `replace-with-random-32-chars` на ваш APP_SECRET

**Вариант B: через SSH (если умеете)**

```bash
cd /volume1/docker/filestash-r2
# скопируйте docker-compose.yml и .env сюда
```

---

## Шаг 4: Создайте .env файл

1. В папке `docker/filestash-r2` скопируйте `.env.example` в новый файл `.env`
2. Откройте `.env` в текстовом редакторе
3. Замените `replace-with-random-32-chars` на ваш секретный ключ (32+ символов)
4. Сохраните файл

---

## Шаг 5: Запустите Filestash через Container Manager

1. Откройте **Container Manager**
2. Перейдите в **Project** (Проект)
3. Нажмите **Create** → **Create from docker-compose**
4. Укажите:
   - **Project name:** filestash-r2
   - **Path:** `/volume1/docker/filestash-r2`
5. Выберите файл `docker-compose.yml`
6. Нажмите **Next** → **Done**
7. Нажмите **Start** для запуска проекта

---

## Шаг 6: Проверьте работу

1. Откройте браузер
2. Введите адрес: `http://IP-ВАШЕГО-NAS:8334`
   (например: `http://192.168.1.100:8334`)
3. Должен открыться Filestash
4. При первом входе задайте пароль администратора
5. Перейдите в **Admin** → **Storage** и настройте подключение к R2 (см. README)

---

## Возможные проблемы

**Контейнер не запускается**
- Проверьте, что в папке есть файл `.env` с APP_SECRET
- В Container Manager → Project → filestash-r2 → Logs посмотрите ошибки

**Страница не открывается**
- Убедитесь, что проект запущен (Status: Running)
- Проверьте, что порт 8334 не занят другим приложением
