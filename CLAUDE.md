# CLAUDE.md

Guidance for Claude Code (and any contributor) working in this repo.

## Что это

`selfsteal` — коллекция статических HTML-сайтов-масок (fallback) для **VLESS Reality**.
Сайт видит любой, кто зашёл на домен по HTTPS без валидного прокси-хендшейка. Задача маски —
выглядеть как настоящий сайт и **не палить прокси**.

## Железное правило: ноль внешних запросов

Каждый сайт при загрузке должен делать **ровно один** сетевой запрос — сам HTML-документ.
Любое изменение обязано это сохранять. **Запрещено** добавлять:

- `<link rel="stylesheet">`, `@import`, `@font-face` с внешним `src`, Google Fonts / любые веб-шрифты;
- внешние `<img src="http...">`, `srcset`, `<use href="http...">`, `<image href="http...">`;
- `<script src>` на CDN (unpkg/jsdelivr/cdnjs/…), любые аналитики/трекеры/пиксели;
- `fetch`/`XMLHttpRequest`/`WebSocket`/`navigator.sendBeacon` на внешние хосты;
- `preconnect`/`preload`/`dns-prefetch`/`prefetch` на внешние домены;
- внешний favicon (только inline `data:image/svg+xml,...`).

**Допустимо** (не является сетевым запросом при загрузке): `<a href="https://...">` в навигации/подвале,
метатеги `rel="canonical"`, OpenGraph/Twitter, JSON-LD `schema.org` — это строки, а не загрузки.
Они даже полезны для достоверности.

### Как получают графику вместо картинок

Только **inline SVG** (иллюстрации, логотипы, иконки, «фото», графики, карты) + CSS-градиенты/паттерны.
Никаких растровых картинок, даже base64 — предпочитаем компактный рисованный SVG.

### Шрифты

Только системные стеки, например:
`-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Helvetica, Arial, sans-serif`,
`Georgia, 'Times New Roman', serif`, `'SF Mono', 'Cascadia Code', Consolas, monospace`.

## Конвенции файлов

- Один сайт = одна папка `sites/NN-slug/` с единственным `index.html`.
- Всё внутри одного файла: `<style>` в `<head>`, `<script>` инлайном в конце. Без внешних ассетов, без сборки.
- Документ начинается с `<!DOCTYPE html>`, валидный, семантичный, адаптивный (mobile + desktop), доступный.
- Тексты — реалистичные и конкретные (вымышленные, но правдоподобные). Никакого lorem ipsum и слова «example».

## Проверка перед коммитом

Скан на утечки (должно быть пусто):

```bash
grep -rEn '(src|href)=["'"'"']https?://|@import|url\(\s*["'"'"']?https?://|fonts\.(googleapis|gstatic)|cdn\.|unpkg|jsdelivr|fetch\(|XMLHttpRequest|new Image\(|<script src|<iframe|srcset|@font-face|preconnect|dns-prefetch|<link[^>]+rel=["'"'"']preload' sites/
```

Ожидаемые совпадения — только `<a href="https://...">` и метатеги. Всё остальное — исправить.
Дополнительно: открыть сайт, DevTools → Network → перезагрузить → должен быть 1 запрос.

## Добавить / пересобрать сайт

- Новый сайт: создать `sites/NN-slug/index.html`, следуя правилам выше, добавить строку в таблицу `README.md`
  и запись в массив `sites` в `_preview/gallery.html`.
- Сайты изначально сгенерированы multi-agent workflow на Opus 4.8 (агент сборки → агент дизайн-QA).
  При регенерации сохраняй разнообразие тематик и эстетик.

## Чего не делать

- Не добавлять package.json / build-инструменты / зависимости — проект принципиально без сборки.
- Не деплоить `_preview/gallery.html` как маску (это локальная витрина, а не fallback).
- Не менять правило «ноль внешних запросов» ради удобства.

## Окружение

Windows, основной shell — PowerShell. Git есть; `gh` CLI нет. Пуш по HTTPS через Git Credential Manager
(запись `git:https://github.com` для пользователя `kewldan`). Remote: `origin` → `github.com/kewldan/selfsteal`.
