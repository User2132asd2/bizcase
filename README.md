# Сайт BizCase («Окно №3»)

Статический сайт-визитка игры и юридические страницы. Без сборки, без внешних шрифтов, CDN, cookie и счётчиков.
Работает без JavaScript (`assets/main.js` — только плавная прокрутка, появление блоков, меню на телефоне,
подсветка оглавления).

```
site/
├── index.html          главная (RU)
├── privacy.html        Политика конфиденциальности (RU)
├── terms.html          Условия использования (RU)
└── assets/
    ├── style.css, main.js
    ├── favicon-32.png, apple-touch-icon.png, icon-512.webp
    ├── og-image.jpg    1200×630, превью для соцсетей
    └── shots/*.webp    скриншоты App Store, 640 px по ширине
```

Скриншоты взяты из `~/art/appstore/*.png` (1320×2868) и уменьшены до 640 px (WebP q86, 40–150 КБ). Иконка —
`app-icon-pie-chart.png`. Чтобы пересобрать картинки после новых скриншотов — уменьшите PNG до ширины 640 и
сохраните в `assets/shots/` с теми же именами.

## Перед публикацией: заменить заглушки

Все заглушки помечены атрибутом `data-placeholder` и HTML-комментарием `<!-- TODO: … -->`:

```bash
grep -rn 'data-placeholder' site/ | sed -E 's/.*data-placeholder="([^"]+)".*/\1/' | sort | uniq -c
grep -rn 'TODO:' site/
```

| Заглушка | Где | На что заменить |
|---|---|---|
| `contact-email` | по одному месту на каждой странице: главная — блок «Поддержка» (`#support`), политика и условия — последний раздел «Контакты» (`#contacts`) | адрес поддержки, лучше ссылкой `<a href="mailto:…">…</a>`; убрать класс `placeholder` |
| `developer` | `privacy.html` §1, `terms.html` §1 | юридическое имя разработчика (ФИО ИП / название компании, при необходимости — адрес) |
| `governing-law` | `terms.html` §14 | применимое право и суд, например «Российской Федерации» |
| `appstore-url` | две кнопки на каждой главной (RU/EN) | `href="https://apps.apple.com/app/id…"`; убрать `aria-disabled="true"` (кнопка станет яркой, пропадёт пунктир) и подпись «Скоро в App Store» |
| `site-url` | `<head>` всех страниц: canonical, `og:url`, `og:image`, `twitter:image` | заменить `https://example.com/` на адрес сайта: `sed -i '' 's#https://example.com/#https://ВАШ-АДРЕС/#g' site/*.html` |

Также проверить: если в приложении «Анонимная статистика» управляет и AppsFlyer / Performance Monitoring, или
если что-то из SDK не подключено, — поправить разделы 4–6 и 9 политики (RU и EN).

## Деплой

Подойдёт любой статический хостинг — публикуется содержимое папки `site/` как корень сайта.

**GitHub Pages.** Репозиторий → Settings → Pages → Source: *GitHub Actions* и workflow с
`actions/upload-pages-artifact` (`path: site`), либо отдельная ветка `gh-pages`, в корне которой лежит содержимое
`site/`:

```bash
git subtree push --prefix site origin gh-pages
```

**Netlify.** New site → Deploy manually → перетащить папку `site/`; или `netlify deploy --prod --dir=site`.

**Firebase Hosting.** В корне репозитория `firebase init hosting`, public directory — `site`, SPA — нет;
затем `firebase deploy --only hosting`.

Свой домен и HTTPS настраиваются в панели хостинга. После деплоя заменить `https://example.com/` (см. выше).

## URL для App Store Connect

| Поле | URL |
|---|---|
| Privacy Policy URL (App Information, обязательно) | `https://ВАШ-АДРЕС/privacy.html` |
| Support URL (версия, обязательно) | `https://ВАШ-АДРЕС/#support` (на странице должен быть реальный e-mail) |
| Marketing URL (версия, по желанию) | `https://ВАШ-АДРЕС/` |
| Лицензионное соглашение (App Information → License Agreement, по желанию) | стандартное Apple EULA или свой текст со ссылкой на `https://ВАШ-АДРЕС/terms.html` |


Ответы App Privacy в App Store Connect должны совпадать с политикой: с Firebase и AppsFlyer карточка уже не
«Data Not Collected» — обновить `docs/release/app-privacy.md` §3 (добавить AppsFlyer, Performance Monitoring,
Device ID/IDFA, Advertising Data, трекинг при согласии ATT) и `Resources/PrivacyInfo.xcprivacy`
(`NSPrivacyTracking`, `NSPrivacyTrackingDomains` для доменов AppsFlyer, `NSUserTrackingUsageDescription` в
`Info.plist`).

## Проверка

```bash
cd site && python3 -m http.server 8000   # открыть http://localhost:8000
```
