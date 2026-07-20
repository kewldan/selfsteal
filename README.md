# 🎭 selfsteal

> Самодостаточные HTML-сайты-маски для **VLESS Reality** (self-steal / steal-your-own-domain).

![HTML](https://img.shields.io/badge/HTML-single--file-E34F26?style=flat&logo=html5&logoColor=white)
![Sites](https://img.shields.io/badge/сайтов-10-1E47F0?style=flat)
![External requests](https://img.shields.io/badge/внешних%20запросов-0-0E9B66?style=flat)
![License](https://img.shields.io/badge/license-MIT-yellow?style=flat)

Когда на твой домен заходит браузер, сканер или активный пробер, Reality отдаёт ему обычный
fallback-сайт. Чтобы это выглядело убедительно, сайт должен быть **настоящим на вид** и при этом
**не делать ни одного внешнего запроса** — любая утечка к CDN, шрифтам или аналитике демаскирует прокси.

Здесь 10 таких сайтов. Каждый — **один файл** `index.html`: весь CSS/JS внутри, вся графика — inline SVG,
шрифты системные. Ноль зависимостей, ноль сборки, ноль исходящих запросов при загрузке.

---

## 🗂 Сайты

| # | Папка | Легенда | Эстетика |
|---|-------|---------|----------|
| 01 | [`01-lens-and-light`](sites/01-lens-and-light/) | Фото-портфолио | Тёмная галерея, serif + amber |
| 02 | [`02-ember-roasters`](sites/02-ember-roasters/) | Обжарщики кофе | Тёплая, espresso / terracotta, магазин |
| 03 | [`03-metricly`](sites/03-metricly/) | SaaS-аналитика | Чистая, indigo, SVG-дашборд, pricing |
| 04 | [`04-pixelforge`](sites/04-pixelforge/) | Инди гейм-студия | Неон на тёмном |
| 05 | [`05-sattva-studio`](sites/05-sattva-studio/) | Йога / wellness | Спокойная, sage / sand, расписание |
| 06 | [`06-byte-sized`](sites/06-byte-sized/) | Дев-блог | Минимализм, light/dark, mono |
| 07 | [`07-form-and-function`](sites/07-form-and-function/) | Архитектурное бюро | Editorial, Swiss grid |
| 08 | [`08-nimbus-cloud`](sites/08-nimbus-cloud/) | Cloud status + docs | Технично, «всё OK», uptime |
| 09 | [`09-copper-fork`](sites/09-copper-fork/) | Ресторан / бистро | Элегантная, green / copper, меню |
| 10 | [`10-northwind-studio`](sites/10-northwind-studio/) | Креативное агентство | Смелая, градиенты, кейсы |

Локальное превью всех сразу — открой **[`_preview/gallery.html`](_preview/gallery.html)** в браузере.

---

## ✨ Особенности

- 🔇 **Zero external requests.** Ни CDN, ни Google Fonts, ни внешних картинок/скриптов, ни аналитики, ни `fetch`. При открытии — ровно один запрос (сам документ).
- 📄 **Один файл на сайт.** Копируй `index.html` куда угодно — сразу работает.
- 🖼 **Настоящий вид.** Реалистичные тексты, цены, разделы; вся графика нарисована как inline SVG.
- 📱 **Адаптивность и доступность.** Мобайл + десктоп, семантика, контраст, focus-состояния.
- 🧳 **Никакой сборки и зависимостей.** Нет `node_modules`, нет build-шага.

---

## 🚀 Быстрый старт

Выбери один сайт как маску и отдай его любым статик-сервером. Пример с Caddy (сам берёт Let's Encrypt):

```bash
mkdir -p /var/www/decoy
cp sites/03-metricly/index.html /var/www/decoy/index.html
```

```caddy
# Caddyfile
your-domain.com {
    root * /var/www/decoy
    file_server
    encode gzip
}
```

Затем направь Reality `dest` → локальный веб-сервер, а `serverNames` → твой домен.
Полный гайд (Caddy, nginx, фрагмент `realitySettings` для Xray, sing-box) — в **[DEPLOY.md](DEPLOY.md)**.

Без Reality тоже работает: каждая папка — готовый статический сайт для Netlify / Cloudflare Pages /
GitHub Pages / любого nginx.

---

## 🔍 Проверка «нет внешних запросов»

Открой сайт → DevTools → **Network** → перезагрузи. Должен быть **один** запрос — документ.

Скан по исходникам (не должно быть авто-загружаемых ресурсов):

```bash
grep -rEn '(src|href)=["'"'"']https?://|@import|url\(\s*["'"'"']?https?://|fonts\.(googleapis|gstatic)|cdn\.|fetch\(|<script src|<iframe|srcset|@font-face' sites/
```

Единственные `https://` в разметке — это `<a href>` в меню/подвале (правдоподобные подстраницы)
и метатеги `canonical`/OpenGraph. Они **не загружаются**, пока по ним не кликнут, и лишь усиливают достоверность.

---

## 📁 Структура

```
selfsteal/
├── sites/                 10 готовых сайтов-масок (по одному index.html)
│   ├── 01-lens-and-light/
│   ├── ...
│   └── 10-northwind-studio/
├── _preview/
│   └── gallery.html       локальная галерея для просмотра всех 10
├── DEPLOY.md              деплой + связка с VLESS Reality
└── README.md
```

---

## ⚖️ Дисклеймер

Проект — для легального обхода цензуры и защиты приватности. Используй в рамках законов своей юрисдикции.
Авторы не несут ответственности за нарушения.

## 📄 Лицензия

MIT — см. [LICENSE](LICENSE).
