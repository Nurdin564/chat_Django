# Chat Django - Многофункциональное чат-приложение

Полнофункциональное веб-приложение для общения в реальном времени, разработанное на Django с поддержкой прямых сообщений, групповых чатов, автоматического перевода сообщений и отслеживания активности.

## 🚀 Основные возможности

- **Аутентификация пользователей** - Регистрация и вход в систему
- **Прямые сообщения** - Общение 1 на 1 между пользователями
- **Групповые чаты** - Создание и управление группами, добавление участников
- **Отправка сообщений в реальном времени** - WebSocket поддержка через Django Channels
- **Автоматический перевод сообщений** - Интеграция с Google Cloud Translation API
- **Управление контактами** - Добавление друзей и управление списком контактов
- **Отслеживание активности** - Мониторинг времени активности пользователей в группах
- **Отметка о прочтении** - Отслеживание прочитанных сообщений

## 📋 Требования

- Python 3.8+
- Django 5.1.7
- PostgreSQL или SQLite (по умолчанию)
- Redis (опционально, для production)
- Google Cloud Translation API ключ

## 🛠️ Установка

### 1. Клонирование репозитория

```bash
git clone <repository-url>
cd chat_django
```

### 2. Создание виртуального окружения

```bash
# На Windows
python -m venv venv
venv\Scripts\activate

# На macOS/Linux
python3 -m venv venv
source venv/bin/activate
```

### 3. Установка зависимостей

```bash
pip install -r requirements.txt
```

### 4. Конфигурация переменных окружения

Создайте файл `.env` в корне проекта:

```env
GOOGLE_KEY_PATH=path/to/your/credentials.json
SECRET_KEY=your-secret-key-here
DEBUG=True
ALLOWED_HOSTS=localhost,127.0.0.1
```

**Получение Google Cloud ключа:**
1. Перейдите на [Google Cloud Console](https://console.cloud.google.com/)
2. Создайте новый проект
3. Включите API: Translation API
4. Создайте Service Account и скачайте JSON ключ
5. Укажите путь к ключу в `GOOGLE_KEY_PATH`

### 5. Применение миграций БД

```bash
python manage.py migrate
```

### 6. Создание суперпользователя (администратора)

```bash
python manage.py createsuperuser
```

### 7. Запуск сервера разработки

```bash
# Используя Daphne для поддержки WebSocket
daphne -b 0.0.0.0 -p 8000 chat_django.asgi:application

# Или используя стандартный Django server
python manage.py runserver
```

Приложение будет доступно по адресу: `http://localhost:8000`

## 📁 Структура проекта

```
chat_django/
├── accounts/              # Модуль аутентификации пользователей
│   ├── models.py
│   ├── views.py
│   ├── forms.py
│   ├── urls.py
│   └── migrations/
├── chat/                  # Основной модуль чатов
│   ├── models.py         # Message model
│   ├── views.py          # Представления чатов
│   ├── consumers.py      # WebSocket consumers для real-time
│   ├── routing.py        # WebSocket маршруты
│   ├── utils/
│   │   └── translate.py  # Функции перевода
│   └── migrations/
├── friends/              # Управление контактами
│   ├── models.py         # Friend model
│   ├── views.py
│   └── migrations/
├── groups/               # Управление группами
│   ├── models.py         # Group, GroupMember, GroupActivity models
│   ├── views.py
│   └── migrations/
├── static/               # Статические файлы
│   ├── css/             # Стили приложения
│   ├── js/              # JavaScript код
│   └── image/           # Изображения
├── templates/            # HTML шаблоны
├── chat_django/          # Конфиг проекта Django
│   ├── settings.py
│   ├── urls.py
│   ├── asgi.py          # ASGI конфиг для WebSocket
│   └── wsgi.py
├── manage.py            # Django управление проектом
├── requirements.txt     # Зависимости проекта
└── db.sqlite3          # База данных (development)
```

## 📚 Основные модели

### User (встроенная Django модель)
- Встроенная модель Django для управления пользователями

### Message
- `sender` - Отправитель сообщения
- `recipient` - Получатель (для приватных сообщений)
- `group` - Группа (для групповых сообщений)
- `content` - Содержание сообщения
- `translated_content` - Переведённое содержание
- `is_read` - Статус прочтения
- `created_at` - Время создания
- `updated_at` - Время последнего обновления

### Friend
- `owner` - Пользователь, который добавил контакт
- `contact` - Добавленный контакт
- `created_at` - Время добавления

### Group
- `name` - Название группы
- `owner` - Владелец группы
- `created_at` - Время создания

### GroupMember
- `group` - Группа
- `user` - Участник
- `added_at` - Время добавления

### GroupActivity
- `group` - Группа
- `user` - Пользователь
- `start_time` - Время начала сессии
- `end_time` - Время окончания сессии
- `duration` - Длительность сессии

## 🌐 API Endpoints

### Аутентификация
- `POST /accounts/login/` - Вход в систему
- `POST /accounts/register/` - Регистрация
- `GET /accounts/logout/` - Выход из системы

### Чаты
- `GET /chat/<chat_type>/<chat_id>/` - Получить чат (group или direct)
- `POST /chat/<chat_type>/<chat_id>/` - Отправить сообщение

### Друзья
- `GET /friends/` - Список контактов
- `POST /friends/add/` - Добавить контакт
- `DELETE /friends/<friend_id>/` - Удалить контакт

### Группы
- `GET /groups/` - Список групп
- `POST /groups/create/` - Создать группу
- `GET /groups/<group_id>/` - Информация о группе
- `POST /groups/<group_id>/add-member/` - Добавить участника
- `GET /groups/<group_id>/activity/` - Активность в группе

## 🔧 WebSocket (Real-time)

Приложение использует Django Channels для поддержки WebSocket соединений:

- **URL**: `ws://localhost:8000/ws/chat/<chat_id>/`
- **Автоматическая доставка сообщений** в реальном времени
- **Уведомления** о статусе подключения

## 🔐 Безопасность

- ✅ CSRF защита
- ✅ SQL injection защита (ORM Django)
- ✅ XSS защита
- ✅ Требуется аутентификация для большинства операций
- ⚠️ **Production:** Установить `DEBUG = False` и использовать `SECRET_KEY` из переменных окружения

## 📝 Переменные окружения

```env
# Google Cloud
GOOGLE_KEY_PATH=путь/к/credentials.json

# Django
SECRET_KEY=ваш-секретный-ключ
DEBUG=False                    # Для production
ALLOWED_HOSTS=yourdomain.com,www.yourdomain.com

# База данных (опционально)
DB_ENGINE=django.db.backends.postgresql
DB_NAME=chat_db
DB_USER=postgres
DB_PASSWORD=password
DB_HOST=localhost
DB_PORT=5432
```

## 🚀 Развёртывание на Production

### Используя Heroku

```bash
# Установка Heroku CLI
# https://devcenter.heroku.com/articles/heroku-cli

# Логин в Heroku
heroku login

# Создание приложения
heroku create your-app-name

# Установка переменных окружения
heroku config:set GOOGLE_KEY_PATH=<path>
heroku config:set SECRET_KEY=<your-key>
heroku config:set DEBUG=False

# Развёртывание
git push heroku main
```

### Используя Docker

```dockerfile
FROM python:3.11

WORKDIR /app

COPY requirements.txt .
RUN pip install -r requirements.txt

COPY . .

CMD ["daphne", "-b", "0.0.0.0", "-p", "8000", "chat_django.asgi:application"]
```

## 📦 Зависимости проекта

- **Django** 5.1.7 - Веб-фреймворк
- **Channels** - WebSocket поддержка
- **Daphne** - ASGI сервер
- **google-cloud-translate** - API перевода
- **python-dotenv** - Управление переменными окружения
- **protobuf** - Сериализация данных

## 🐛 Отладка

### Логирование WebSocket
```python
# В settings.py
LOGGING = {
    'version': 1,
    'disable_existing_loggers': False,
    'handlers': {
        'console': {
            'class': 'logging.StreamHandler',
        },
    },
    'loggers': {
        'channels': {
            'handlers': ['console'],
            'level': 'DEBUG',
        },
    },
}
```

### Тестирование
```bash
python manage.py test
```

## 🤝 Вклад

Для внесения изменений:

1. Создайте Fork репозитория
2. Создайте ветку для вашей фичи (`git checkout -b feature/AmazingFeature`)
3. Коммитьте изменения (`git commit -m 'Add some AmazingFeature'`)
4. Отправьте в ветку (`git push origin feature/AmazingFeature`)
5. Откройте Pull Request

## 📄 Лицензия

Этот проект лицензирован под MIT License. Смотрите файл `LICENSE` для деталей.

## 📧 Контакты

Для вопросов и предложений свяжитесь с разработчиком через Issues на GitHub.

## 🔗 Полезные ссылки

- [Django Документация](https://docs.djangoproject.com/)
- [Django Channels](https://channels.readthedocs.io/)
- [Google Cloud Translation](https://cloud.google.com/translate/docs)
- [WebSocket Protocol](https://tools.ietf.org/html/rfc6455)

---

**Версия:** 1.0.0  
**Статус:** Development  
**Последнее обновление:** Июнь 2026
