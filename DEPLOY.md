# VLess Reality decoy sites

10 самодостаточных статических сайтов-масок. Каждый — один файл `sites/NN-name/index.html`,
без единого внешнего запроса (шрифты системные, вся графика — inline SVG, весь CSS/JS внутри файла).
Это то, что видит любопытный человек или сканер, зашедший на твой домен по HTTPS.

| # | Папка | Легенда |
|---|-------|---------|
| 01 | `01-lens-and-light`   | Фото-портфолио |
| 02 | `02-ember-roasters`   | Обжарщики кофе |
| 03 | `03-metricly`         | SaaS-аналитика (лендинг) |
| 04 | `04-pixelforge`       | Инди гейм-студия |
| 05 | `05-sattva-studio`    | Йога / wellness |
| 06 | `06-byte-sized`       | Дев-блог |
| 07 | `07-form-and-function`| Архитектурное бюро |
| 08 | `08-nimbus-cloud`     | Cloud status + docs |
| 09 | `09-copper-fork`      | Ресторан / бистро |
| 10 | `10-northwind-studio` | Креативное агентство |

Просмотреть все локально: открой `_preview/gallery.html` в браузере.

---

## Как это работает с Reality

Reality не отдаёт HTML сам. Твой прокси (Xray / sing-box) на «неправильном» рукопожатии
(браузер, сканер, активный пробер) **отправляет клиента на `dest`** — обычный TLS-сайт.
Чтобы маскировкой был *твой* сайт, а не чужой: подними локальный веб-сервер с этим сайтом
и настоящим сертификатом на свой домен, и направь `dest` на него.

```
Браузер / пробер ─┐
                  ├─► :443 Xml/sing-box (Reality)
Клиент-прокси   ─┘        │
                          ├─ валидный VLESS  → проксируется
                          └─ всё остальное   → dest → 127.0.0.1:8443 (nginx/caddy с сайтом)
```

---

## 1. Отдать сайт веб-сервером

### Caddy (проще всего — сам берёт Let's Encrypt)

Выбери один сайт как маску и положи его в корень. `Caddyfile`:

```caddy
your-domain.com {
    root * /var/www/decoy          # сюда скопирован index.html выбранного сайта
    file_server
    encode gzip
}
```

```bash
mkdir -p /var/www/decoy
cp sites/03-metricly/index.html /var/www/decoy/index.html
caddy run --config Caddyfile
```

Caddy слушает 443 с реальным сертификатом. Reality `dest` → `127.0.0.1:8443`
(если 443 занят прокси — переставь Caddy на 8443 и `tls internal` либо сними cert отдельно).

### nginx

```nginx
server {
    listen 8443 ssl http2;
    server_name your-domain.com;

    ssl_certificate     /etc/letsencrypt/live/your-domain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/your-domain.com/privkey.pem;

    root /var/www/decoy;
    index index.html;
    location / { try_files $uri $uri/ =404; }
}
```

```bash
cp sites/09-copper-fork/index.html /var/www/decoy/index.html
nginx -t && systemctl reload nginx
```

---

## 2. Направить Reality на этот сайт (Xray)

Фрагмент `realitySettings` в inbound:

```jsonc
"streamSettings": {
  "security": "reality",
  "realitySettings": {
    "show": false,
    "dest": "127.0.0.1:8443",          // локальный nginx/caddy с сайтом
    "serverNames": ["your-domain.com"], // = server_name сервера выше
    "privateKey": "<xray x25519 privateKey>",
    "shortIds": ["<hex shortId>"]
  }
}
```

sing-box — аналогично: `tls.reality.handshake.server` → тот же локальный веб-сервер.

Ключевое: `serverNames` = домен на сертификате веб-сервера, `dest`/`handshake` = адрес,
где реально отдаётся сайт. Тогда любой, кто откроет `https://your-domain.com` в браузере,
увидит обычный сайт — а не признак прокси.

---

## 3. Просто статик-хостинг (без Reality)

Каждая папка — готовый сайт: залей `index.html` на Netlify / Cloudflare Pages / GitHub Pages /
S3+CDN / любой nginx. Ничего собирать не нужно, зависимостей нет.

---

## Проверка «нет внешних запросов»

Открой сайт в браузере → DevTools → **Network** → перезагрузи.
Должен быть ровно **один** запрос (сам документ). Ни шрифтов, ни картинок, ни аналитики.
Все `https://`-ссылки в разметке — это `<a href>` в меню/подвале (ведут на правдоподобные
подстраницы) и метатеги `canonical`/OpenGraph: они **не загружаются**, пока по ним не кликнут.
