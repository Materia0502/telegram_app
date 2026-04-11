# Telegram Mini App — Оплата по QR (СБП)

Сканирует QR-коды Системы быстрых платежей (СБП) и отправляет ссылку на сервер оплаты.

## Файлы

```
tg-mini-app/
└── index.html   — всё приложение в одном файле
```

## Подключение к Telegram-боту

### 1. Создать бота через @BotFather

Откройте [@BotFather](https://t.me/BotFather) в Telegram и выполните:

```
/newbot
```

Следуйте инструкциям: задайте имя и username бота. Сохраните полученный `BOT_TOKEN`.

---

### 2. Захостить index.html

Файл должен быть доступен по HTTPS. Варианты:

| Хостинг | Команда / способ |
|---|---|
| **GitHub Pages** | Push в репо → Settings → Pages → выбрать ветку |
| **Vercel** | `npx vercel --prod` в папке tg-mini-app/ |
| **Netlify** | Перетащить папку на netlify.com/drop |
| **Cloudflare Pages** | `npx wrangler pages deploy .` |

Пример итогового URL: `https://your-app.vercel.app/`

---

### 3. Создать Mini App через @BotFather

```
/newapp
```

BotFather спросит:
1. **Выберите бота** — выберите созданного бота
2. **Title** — название приложения, например `QR Pay`
3. **Description** — краткое описание
4. **Photo** — фото 640×360 px (можно пропустить)
5. **Web App URL** — вставьте ваш HTTPS-URL с index.html

После этого BotFather выдаст прямую ссылку на Mini App вида:
```
https://t.me/YourBot/qrpay
```

---

### 4. Добавить кнопку в бота (опционально)

Чтобы пользователи открывали Mini App прямо из чата, добавьте `InlineKeyboardButton` с `web_app`:

```python
from telegram import InlineKeyboardButton, InlineKeyboardMarkup, WebAppInfo

button = InlineKeyboardButton(
    text="Оплатить по QR",
    web_app=WebAppInfo(url="https://your-app.vercel.app/")
)
reply_markup = InlineKeyboardMarkup([[button]])
await update.message.reply_text("Нажмите для оплаты:", reply_markup=reply_markup)
```

---

## Требования к серверу

Mini App отправляет POST-запрос:

```http
POST /payment/pay
Content-Type: application/json

{ "qrUrl": "https://qr.nspk.ru/..." }
```

Сервер должен вернуть `2xx` при успехе.

## Как это работает

1. Пользователь нажимает **Сканировать QR**
2. Браузер запрашивает доступ к камере
3. Каждые 200 мс кадр анализируется библиотекой [jsQR](https://github.com/cozmo/jsQR)
4. При нахождении URL с `qr.nspk.ru` — отправляется POST на сервер
5. Показывается результат оплаты; кнопка **Готово** закрывает Mini App
