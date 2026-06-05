# Технічний аудит сайту «Геометрія саду» — 2026-06-05

## Контекст

Аудит виконано для поточного production-файлу `index.html`. За брифом `index.html` є головною сторінкою, а `geometriya-sadu-VANILLA.html` — редіректом на неї. Окремо перевірено секцію `#why`, бо в ній є складна пінована сцена зі сферою на 2D-canvas.

## Команди та перевірки

- `find .. -name AGENTS.md -print && git status --short --branch && git log --oneline --decorate -5 && find . -maxdepth 2 -type f | sed 's#^./##' | sort | head -120`
- `python3`-скрипт для підрахунку розміру `index.html`, локальних asset references, duplicate IDs, placeholder links і debug-слідів.
- `rg -n "href=\"#\"|TODO|FIXME|console\.log|debugger|emailjs|rel=\"canonical\"|meta name=\"description\"|og:|twitter:" index.html geometriya-sadu-VANILLA.html BRIEF-FOR-NEW-CHAT.md PROJECT-AUDIT-2026-06-03.md`
- `du -h index.html images/* videos/* vendor/* fonts/* | sort -hr | head -40`
- `image-size` через Node для перевірки розмірів зображень.
- `python3 -m http.server 4173`
- Playwright + axe-core: desktop `1440x900` і mobile `390x844`.
- `npx html-validate /workspace/geometriya-sadu/index.html`
- Lighthouse через Playwright Chromium: `performance, accessibility, best-practices, seo`.
- `node --check /tmp/index-scripts.js` для синтаксису inline-скриптів.

## Загальна оцінка

Сайт уже має сильну базу: одна production-сторінка, локальні assets, SEO/OG/Twitter metadata, JSON-LD LocalBusiness, один H1, відсутні duplicate IDs, немає `console.log`/`debugger`, а JS-синтаксис inline-скриптів валідний. Основні ризики зараз не в «поламаній сторінці», а в конверсії, доступності, продуктивності та maintainability через великий монолітний HTML/CSS/JS.

## Результати інструментів

### Lighthouse

- Performance: **43/100**
- Accessibility: **95/100**
- Best Practices: **100/100**
- SEO: **100/100**
- LCP: **6.9 s**
- Total Blocking Time: **1,610 ms**
- CLS: **0**
- Speed Index: **4.4 s**
- FCP: **2.8 s**
- Потенційна економія: responsive images — **135 KiB**, offscreen images — **282 KiB**, unused JS — **24 KiB**.

### Playwright + axe-core

- Desktop і mobile відкриваються без JS page errors.
- `h1`: **1**.
- Placeholder links: **5**.
- Axe violations:
  - `aria-allowed-role`: 8 nodes, minor.
  - `color-contrast`: 1 node, serious.
  - `region`: 38–39 nodes, moderate.

### html-validate

`html-validate` знайшов **60 errors**. Частина з них style-policy або рекомендаційні, але є реальні accessibility/HTML-risk категорії: `hidden-focusable`, `no-implicit-button-type`, `aria-label-misuse`, `wcag/h30`, `multiple-labeled-controls`, `element-permitted-content`, `prefer-native-element`, `unique-landmark`.

### Assets

Найважчі файли:

- `videos/services-bg.mp4` — 876 KiB.
- `images/Heroback 2.webp` — 692 KiB.
- `images/Heroback3.webp` — 580 KiB.
- `images/Backv5.webp` — 492 KiB.
- `index.html` — 180 KiB.
- `vendor/gsap.min.js` — 72 KiB.
- `vendor/ScrollTrigger.min.js` — 44 KiB.

## Пріоритетні знахідки

### P0 — Конверсія: placeholder-посилання залишають користувача в нікуди

У футері Instagram, Telegram, WhatsApp і Privacy Policy мають `href="#"`; FAB Telegram також веде на `#`. Це напряму впливає на заявки: людина натискає канал звʼязку, але не переходить у месенджер/соцмережу.

**Рекомендація:** замінити всі заглушки на реальні URL або тимчасово приховати канали, для яких немає посилання. Мінімум: Telegram у FAB не має бути `href="#"`, бо це швидка дія.

### P1 — Performance: низький Lighthouse Performance через LCP/TBT

Lighthouse показує Performance 43/100, LCP 6.9 s і TBT 1,610 ms. На сторінці багато анімацій, великий inline CSS/JS, GSAP/ScrollTrigger/Lenis і важкі фон/картинки. Поточний стан візуально ефектний, але на слабших телефонах може відчуватися як затримка до взаємодії.

**Рекомендації:**

1. Винести частину inline JS/CSS у окремі файли з кешуванням.
2. Розділити ініціалізацію анімацій: critical hero — одразу, інші секції — через IntersectionObserver/lazy init.
3. Перевірити, чи `services-bg.mp4` потрібен на mobile; якщо ні — вимикати або замінити poster.
4. Для великих WebP зробити mobile-sized variants і `srcset`/`picture` там, де це `<img>` або CSS background через media queries.

### P1 — Accessibility/HTML: мобільне меню має focusable links всередині `aria-hidden="true"`

`html-validate` підсвітив `hidden-focusable`: у мобільному меню контейнер має `aria-hidden="true"`, але всередині залишаються посилання. Це може заважати screen reader/focus navigation.

**Рекомендація:** коли меню закрите, робити links inert/`tabindex="-1"`, або використовувати `inert` на контейнері та синхронно прибирати його при відкритті. Також додати `type="button"` для burger button.

### P1 — Forms: нестандартний custom select вкладений у `<label>`

У модальній формі custom select із `div role="listbox"` і buttons вкладений у `<label>`. `html-validate` показує `multiple-labeled-controls`, `element-permitted-content`, `prefer-native-element`. Це ризик для accessibility, screen readers і фокусу.

**Рекомендація:** винести custom select за межі `<label>`, зробити явний текст label через `id` + `aria-labelledby`, або на mobile перейти на native `<select>`.

### P1 — Privacy/Legal: згода на персональні дані є, але політика конфіденційності — заглушка

У CTA є текст про згоду на обробку персональних даних, але Privacy Policy у футері веде на `#`. Це юридично слабке місце й поганий сигнал довіри.

**Рекомендація:** додати сторінку/модалку політики або хоча б окремий HTML-файл `privacy.html`, і привʼязати посилання з CTA/футера.

### P2 — #why: resize сфери виправлено, але секція залишається найризиковішою з точки зору адаптивності

Поточний код canvas сфери вже перемірює host, `devicePixelRatio`, `visualViewport` і `ResizeObserver`, та робить `ScrollTrigger.refresh()` після resize. Це закриває конкретний баг із «сфера не повертається в розміри». Але сама секція `#why` усе ще має складну комбінацію pinned ScrollTrigger + snap + canvas + global keyboard handler.

**Рекомендації:**

1. Зробити окремий regression-тест для `#why`: desktop → mobile → desktop, scroll in/out, snap, resize, orientation change.
2. Обмежити keyboard handler з `window` до активної секції/фокусу, щоб стрілки не перехоплювались несподівано.
3. У майбутньому розглянути `gsap.matchMedia()` для окремого lifecycle desktop/mobile, якщо знову зʼявляться адаптивні edge cases.

### P2 — Accessibility: 8 ticks у `#why` мають ARIA role issue

Ticks створені як `<span>` і JS додає `role="button"`, `tabindex` та aria-label. Axe/html-validate підсвічують роль як неідеальну для елемента.

**Рекомендація:** замінити ticks на реальні `<button type="button" class="gs-why-tick">`, це прибере role workaround і спростить клавіатуру.

### P2 — Footer social links мають SVG-only content без видимого тексту

`html-validate` показує `wcag/h30` для соціальних links. `title` не є достатньою заміною нормального accessible name у всіх сценаріях.

**Рекомендація:** додати `aria-label="Instagram"`, `aria-label="Telegram"`, `aria-label="WhatsApp"` і реальні URL.

### P2 — Image optimization: є oversized зображення для mobile

Багато зображень мають 896×1200 або більше, а деякі background images — 2752×1536. Для desktop це прийнятно, але на mobile зайве. Lighthouse окремо показав економію на responsive/offscreen images.

**Рекомендація:** зробити `-mobile.webp`/`-tablet.webp` варіанти для hero/services/portfolio й підключати через media queries або `<picture>`.

### P2 — Maintainability: `index.html` — великий моноліт

`index.html` має приблизно 180 KiB і 2623 рядки. CSS, HTML і JS в одному файлі швидко ускладнюють регресії, особливо для `#why`, portfolio, forms і modal.

**Рекомендація:** без зміни дизайну рознести:

- `assets/css/main.css`
- `assets/js/forms.js`
- `assets/js/portfolio.js`
- `assets/js/why.js`
- `assets/js/ui.js`

Після цього легше додавати regression checks і кешувати assets.


### P3 — SEO hygiene: немає `robots.txt` / `sitemap.xml`

Lighthouse SEO дає 100/100, але під час локального Lighthouse-прогону був запит до `/robots.txt`, який повернув 404 на локальному сервері. Для GitHub Pages це не блокує індексацію, але краще додати `robots.txt` і `sitemap.xml`, щоб явно вказати canonical URL та майбутні сторінки, якщо зʼявиться `privacy.html`.

**Рекомендація:** додати мінімальний `robots.txt` із `Sitemap: https://yaladnich.github.io/geometriya-sadu/sitemap.xml` і sitemap для головної/політики.

### P3 — html-validate style-policy warnings

Є inline styles для animation delays, honeypot і success states. Це не критично для запуску, але додає шум у валідаторі.

**Рекомендація:** поступово замінити inline styles на класи (`is-hidden`, CSS custom properties для delays, visually-hidden honeypot class).

## Що вже добре

- SEO metadata і Open Graph/Twitter tags присутні.
- JSON-LD LocalBusiness присутній.
- Один H1.
- Duplicate IDs не знайдено.
- Локальні assets загалом на місці; явних битих локальних refs у статичній розмітці не виявлено.
- Нема `console.log`, `debugger`, `TODO/FIXME` в production `index.html`.
- Playwright desktop/mobile не зафіксував JS page errors.
- `node --check` inline-скриптів пройшов.
- CLS за Lighthouse — 0.

## Рекомендований порядок робіт

1. **Швидкий конверсійний hotfix:** реальні social/FAB links + privacy link.
2. **Accessibility hotfix:** `aria-hidden`/focus у mobile menu, `type="button"`, aria-label для SVG-only links.
3. **Форми:** поправити custom select семантику й policy link.
4. **Performance pass:** lazy init важких анімацій, mobile images, video strategy.
5. **#why regression:** автоматизувати resize/orientation/snap тест.
6. **Рефактор моноліту:** винести CSS/JS у модулі без зміни UI.
7. **SEO hygiene:** додати `robots.txt` / `sitemap.xml` після появи privacy page.
