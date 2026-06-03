# Аудит проєкту «Геометрія саду» — 2026-06-03

## Контекст

Перевірено актуальний стан `origin/main` на коміті:

```text
4a0e1c7 test(sphere): щільніша й видима сітка (LINK 0.34, яскравіші ребра)
```

Цей файл створено як окремий артефакт аудиту, щоб не змішувати висновки з робочим брифом і не губити список пріоритетних виправлень у чаті.

---

## Короткий висновок

Критичної синтаксичної поломки під час перевірки не виявлено: репозиторій синхронізується з `origin/main`, GitHub Pages workflow існує, основний HTML має робочу структуру, а секція `#why` уже використовує Three.js/module-import під актуальні візуальні експерименти.

Але в проєкті є технічні й продуктові проблеми, які варто виправити перед наступними великими візуальними змінами:

1. мобільне меню не синхронізоване з desktop-навігацією;
2. у футері та FAB залишилися `href="#"` заглушки;
3. GitHub Pages публікує весь корінь репозиторію, включно з backup/preview-файлами;
4. workflow досі має стару службову Claude-гілку в production deploy trigger;
5. SEO-метадані мінімальні;
6. зображення нижніх секцій потребують `loading="lazy"` / `decoding="async"`;
7. кастомний select і картки послуг потребують accessibility-допрацювання;
8. Three.js-візуал у `#why` потребує fallback/performance-стратегії.

---

## P1 — виправити першими

### 1. Mobile navbar відстає від desktop navbar

У desktop-навігації є пункт `Контакти`, а `FAQ` перейменовано на `Поширені питання`. У mobile-навігації досі залишився короткий `FAQ`, а пункту `Контакти` немає.

Поточний mobile-набір:

```html
<a href="#services" class="gs-mobile-nav-link">Послуги</a>
<a href="#why" class="gs-mobile-nav-link">Переваги</a>
<a href="#portfolio" class="gs-mobile-nav-link">Портфоліо</a>
<a href="#steps" class="gs-mobile-nav-link">Етапи</a>
<a href="#pricing" class="gs-mobile-nav-link">Ціни</a>
<a href="#faq" class="gs-mobile-nav-link">FAQ</a>
```

Рекомендована правка:

```html
<a href="#faq" class="gs-mobile-nav-link">Поширені питання</a>
<a href="#cta" class="gs-mobile-nav-link">Контакти</a>
```

**Ефект:** покращує UX на мобільних без зміни дизайну.

---

### 2. Placeholder-посилання `href="#"`

Залишилися заглушки:

- Instagram у футері;
- Telegram у футері;
- WhatsApp у футері;
- політика конфіденційності;
- Telegram у FAB.

Це впливає на заявки: користувач натискає канал звʼязку, але не потрапляє в месенджер/соцмережу.

Рекомендації:

```html
<!-- WhatsApp -->
<a href="https://wa.me/380683782003" ...>

<!-- Telegram -->
<a href="https://t.me/USERNAME" ...>

<!-- Instagram -->
<a href="https://www.instagram.com/USERNAME/" ...>
```

Для політики конфіденційності потрібно або створити окрему сторінку, або прибрати placeholder-link до появи документа.

---

### 3. GitHub Pages публікує весь корінь репозиторію

Поточний workflow використовує:

```yaml
- name: Upload artifact
  uses: actions/upload-pages-artifact@v3
  with:
    path: .
```

Через це в production artifact потрапляють:

- backup HTML;
- preview/demo HTML;
- експериментальні сторінки;
- великі відео та службові файли.

Ризики:

- старі сторінки можуть відкриватися за прямими URL;
- backup/preview можуть індексуватися;
- artifact більший, ніж потрібно;
- складніше контролювати production-вміст.

Рекомендація:

1. створити production-папку, наприклад `public/`;
2. складати туди тільки файли сайту;
3. змінити Pages workflow на `path: public`;
4. backup/preview залишати поза production artifact.

---

### 4. У workflow залишилася стара Claude-гілка

Production deploy досі запускається з:

```yaml
branches:
  - main
  - claude/blissful-faraday-y2CrU
```

Рекомендація:

```yaml
branches:
  - main
```

**Ефект:** знижує ризик випадкового деплою неактуальної версії сайту.

---

## P2 — SEO, продуктивність, заявки

### 5. SEO-метадані потрібно розширити

У головному HTML є базові `title` і `description`, але варто додати:

- absolute canonical URL;
- Open Graph;
- Twitter Card;
- favicon / apple-touch-icon;
- JSON-LD для локального бізнесу;
- preload hero-зображення;
- `display=swap` у Google Fonts URL.

Мінімальний приклад:

```html
<link rel="canonical" href="https://yaladnich.github.io/geometriya-sadu/">
<meta property="og:title" content="Геометрія саду — Ландшафтний дизайн під ключ у Києві">
<meta property="og:description" content="Ландшафтний дизайн під ключ у Києві та Київській області. Рулонний газон, автополив, озеленення, земляні роботи.">
<meta property="og:image" content="https://yaladnich.github.io/geometriya-sadu/images/Heroback3.webp">
<meta property="og:type" content="website">
<meta property="og:locale" content="uk_UA">
```

---

### 6. Зображення нижніх секцій потребують lazy-loading

Для зображень, які не є критичними для першого екрану, варто додати:

```html
loading="lazy" decoding="async"
```

Це особливо актуально для карток послуг, портфоліо та декоративних зображень нижче hero.

---

### 7. Форми можна посилити HTML-атрибутами

JS-валидація вже є, але полям варто додати:

- `required`;
- `autocomplete`;
- `inputmode="tel"` для телефону;
- `aria-describedby` для помилок.

Приклад:

```html
<input name="name" autocomplete="name" required aria-describedby="cta-name-err">
<input type="tel" name="phone" autocomplete="tel" inputmode="tel" required aria-describedby="cta-phone-err">
```

---

## P3 — доступність і якість коду

### 8. Кастомний select неповністю доступний

Потрібно додати:

- `aria-selected="true/false"` для опцій;
- фокус активної опції при відкритті;
- клавіатурну навігацію `ArrowUp` / `ArrowDown`;
- вибір через `Enter` / `Space`;
- закриття через `Escape`.

---

### 9. Сервісні картки клікабельні через `<article onclick>`

Для миші це працює, але для клавіатури й screen reader семантика слабка.

Мінімальна правка:

```html
<article class="gs-pcard" role="button" tabindex="0">
```

Також потрібен JS-обробник для `Enter` і `Space`.

Кращий варіант — зробити інтерактивний елемент справжньою кнопкою або посиланням.

---

### 10. Не-submit кнопкам додати `type="button"`

Усі кнопки, які не мають відправляти форму, краще явно позначити:

```html
<button type="button">
```

Це запобігає випадковим submit-поведінкам після майбутніх змін DOM.

---

## P4 — Three.js `#why` і технічна гігієна

### 11. `#why` потребує fallback/performance-стратегії

Поточний напрям із Three.js виглядає як основний візуальний експеримент, але він має ризики:

- залежність від CDN;
- WebGL може бути важким для слабких телефонів;
- bloom/postprocessing можуть давати просідання FPS;
- потрібен fallback, якщо module script не завантажився;
- бажано вимикати або спрощувати ефект при `prefers-reduced-motion`.

Рекомендації:

1. додати статичний fallback-фон;
2. вимикати важкий WebGL при `prefers-reduced-motion`;
3. протестувати на реальних Android/iPhone;
4. зафіксувати фінальний стан `#why` у брифі.

---

### 12. Cleanup backup/preview/важких файлів

У репозиторії вже є кілька backup/preview HTML і великі медіафайли. Якщо Pages продовжує деплоїти `path: .`, вони публікуються разом із сайтом.

Рекомендації:

- не деплоїти backup/preview/prototype файли;
- перемістити production-сайт у `public/`;
- видалити або винести невикористані MP4;
- залишити в `main` тільки те, що потрібно для production або документації.

---

## Рекомендований порядок робіт

### Етап 1 — безпечний cleanup без зміни дизайну

1. Синхронізувати mobile navbar з desktop navbar.
2. Прибрати стару Claude-гілку з Pages workflow.
3. Замінити відомі placeholder-посилання.
4. Додати `type="button"` до не-submit кнопок.
5. Додати `loading="lazy" decoding="async"` для нижніх зображень.
6. Додати canonical/Open Graph meta.

### Етап 2 — деплой-гігієна

1. Винести production-файли в `public/`.
2. Налаштувати Pages workflow на `path: public`.
3. Прибрати backup/preview/prototype з production artifact.

### Етап 3 — accessibility і форми

1. Підсилити HTML-атрибути форм.
2. Додати доступність custom select.
3. Зробити сервісні картки доступними з клавіатури.
4. Додати сторінку політики конфіденційності.

### Етап 4 — стабілізація `#why`

1. Перевірити Three.js-візуал на реальних телефонах.
2. Додати fallback.
3. Додати reduced-motion режим.
4. Оновити бриф під фактичний фінальний стан.

---

## Перевірки, на яких базується аудит

```bash
git fetch origin main --quiet
git reset --hard origin/main
git rev-list --left-right --count HEAD...origin/main
git log --date=iso-strict --pretty=format:'%h %ad %s' -18
rg -n "gs-mobile-nav-link|href=\"#\"|upload-pages-artifact|claude/blissful|three|importmap|loading=|og:|canonical" geometriya-sadu-VANILLA.html .github/workflows/pages.yml index.html BRIEF-FOR-NEW-CHAT.md
git ls-files '*backup*' '*preview*' 'dots-*.html'
```
