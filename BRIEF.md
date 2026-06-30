# Геометрія саду — Бриф проєкту

Лендінг ландшафтного дизайну. Один файл `index.html` (vanilla HTML/CSS/JS, без збірки).
Деплой: **GitHub Pages** з гілки `main` → https://yaladnich.github.io/geometriya-sadu/

---

## ⚠️ ВАЖЛИВО: Коли додаєш нове фото

Для **кожного** нового зображення, що показується на сторінці, треба робити **стиснуту мобільну версію**.
Без цього мобільні користувачі вантажать важкі десктопні фото → повільний сайт, падає PageSpeed.

### Правило іменування
- Десктоп: `назва.webp`
- Мобільна: `назва-m.webp` (той самий файл + суфікс `-m`)
- **Без пробілів** у назві `-m` файлу (пробіли ламають `srcset`) — заміняй пробіли на `_`
  - приклад: `Рулонний газон.webp` → `Рулонний_газон-m.webp`

### Як стиснути (Python + Pillow)
```python
from PIL import Image
img = Image.open('images/назва.webp')
w, h = img.size
MAX_W = 768            # ширина для мобільного
if w > MAX_W:
    img = img.resize((MAX_W, int(h*MAX_W/w)), Image.LANCZOS)
img.save('images/назва-m.webp', 'WEBP', quality=72, method=6)
```
- Якість `72` — норма. Для повноекранних фонів із затемненням можна `58`.
- Портретні фото (висота > ширини): зменшуй до ~700px ширини, бо вони й так вузькі.

### Як підключити в HTML
**Картинка `<img>`** → обгорни в `<picture>`:
```html
<picture>
  <source media="(max-width:768px)" srcset="images/назва-m.webp">
  <img src="images/назва.webp" alt="...">
</picture>
```

**CSS-фон** → media query:
```css
@media(max-width:768px){
  .селектор{background-image:url('images/назва-m.webp');}
}
```

**Фон через JS (портфоліо)** → додай масив `imagesMobile` поряд з `images`.

**Preload у `<head>`** (для хіро) → умовний через inline-скрипт:
```html
<script>document.write('<link rel="preload" href="images/'+(window.innerWidth<=768?'назва-m':'назва')+'.webp" as="image">');</script>
```

---

## Структура секцій
- **Hero** — `Backv5.webp` фон (мобільний `Backv5-m.webp`)
- **Послуги** (`#services`) — 8 карток `.gs-pcard`, кожна з `<picture>`
- **Why** (`#why`) — слайдер: десктоп = GSAP ScrollTrigger-пін, мобільний = нативний CSS scroll-snap (див. ⚠️ Інваріанти)
- **Портфоліо** (`#portfolio`) — слайдер фонів, масиви `portfolioProjects[].images` / `.imagesMobile`
- **Ціни** (`#pricing`), **FAQ** (`#faq`), **Footer**

## ⚠️ Інваріанти слайдерів — НЕ ЛАМАТИ при правках

Обидва слайдери крихкі до гонок і конфлікту з браузерним скролом. Перед правкою прочитай це.

### Слайдер «Переваги» (#why, `initWhyPin`, ~рядок 2549)
- **Дві РІЗНІ реалізації за типом пристрою — НЕ об'єднувати:**
  - **Десктоп (`pointer:fine`)**: ScrollTrigger-пін кадру + ручний wheel-крок. Lenis віртуалізує
    скрол і має ВЛАСНИЙ wheel-слухач → крокувати лише через `lenis.scrollTo({lock:true,force:true})`,
    а сам wheel глушити в `capture`-фазі + `stopImmediatePropagation()` (інакше Lenis накопичує інерцію
    і прокручує кілька слайдів повз крок).
  - **Мобільний (`pointer:coarse`)**: НАТИВНИЙ CSS `scroll-snap` (клас `gs-why-snap`), БЕЗ піна й БЕЗ
    JS-хайджеку тача. Гілка робить `return` до створення ScrollTrigger. Активний слайд ловить
    `IntersectionObserver`. **НЕ повертати JS-перехоплення `touchmove` на мобільному** — compositor
    стартує нативний скрол раніше за `preventDefault`, і «один змах = один слайд» зривається.
- **Один жест = один слайд**: крок ведеться по `target` (індекс-ціль), НЕ по `cur` (відрендерений слайд).
  Розблокування жесту — лише коли колесо мовчить ≥160мс **І** `stepTween` завершено.
- **`stepId`-токен**: тільки онкомпліт НАЙновішого кроку скидає `stepTween` (старий Lenis-scrollTo,
  скасований новим, інакше розблокує посеред кроку).
- **`isPinned()`** звіряється з діапазоном `scrollY`, бо `st.isActive` буває `undefined` на `progress=0`.
- НЕ додавати вбудований ScrollTrigger `snap` — конкурує з ручним кроком, дає flicker «через слайд і назад».
- CSS `gs-why-snap`: `scroll-snap-type:y mandatory` + `overscroll-behavior-y:contain` (тримає «пастку»
  секції). Сфера `.gs-why-dots` — `sticky` з `margin-bottom:-40svh`. Панелі `height:100svh; scroll-snap-align:start`.

### Слайдер портфоліо (#portfolio, `updatePortfolio`/`pfGo`, ~рядок 1607)
- **Один клік = одне зображення**: новий клік під час кросфейду (700мс) МИТТЄВО доклацує попередній
  перехід через `pfFinish()`, а не ігнорується. **НЕ повертати `if (portfolioTransitioning) return`**
  у `pfGo`/`pfSelectProject` — на мобільному тапи швидші за 700мс і кожен 2-3 губився б.
- `pfFinish` тримає завершувач поточного кросфейду; `finish()` чистить `finishTimer` і обнуляє `pfFinish`.
- `portfolioTransitioning` лишається як прапор стану, але вже НЕ блокує нові кліки.

---

## Технічне
- Шрифт: **Onest** (локальний `fonts/`), один на весь сайт
- Анімації: **GSAP + ScrollTrigger** (`vendor/`) — підключені БЕЗ `defer` (defer ламає hero-анімації)
- Форма: **Web3Forms** (fetch POST, без зовнішніх скриптів), honeypot анти-спам
- Git: автор комітів `Claude <noreply@anthropic.com>`, гілка розробки `claude/busy-cerf-uH3lP` → `main`

## Структура index.html (рядки)
| Блок | Рядки | Зміст |
|---|---|---|
| `<head>` | 1–50 | meta, preload, шрифти. Inline `<script>` з `document.write` на рядку ~11 — **не чіпати** |
| `<style>` | 51–900 | весь CSS. Префікс класів `gs-`. `:root` змінні. Брейкпоінт `768px` |
| `<body>` | 900–1445 | Hero → Services → Why → Portfolio → Pricing → FAQ → CTA → Footer → Modal |
| `<script>` | 1445–2729 | весь JS. `gsap.min.js` + `ScrollTrigger.min.js` без `defer` на рядках 1446–1447 |

## Ключові JS-функції та змінні

| Символ | Рядок | Призначення |
|---|---|---|
| `WEB3FORMS_KEY` | ~1466 | Публічний ключ Web3Forms |
| `portfolioProjects[]` | ~1469 | Масив проєктів з `images[]` та `imagesMobile[]` |
| `submitForm(event, prefix)` | ~1801 | Відправка форми; `prefix` = `'cta'` або `'modal'` |
| `openModal(service?)` | ~1673 | Відкриває модалку, опціонально обирає послугу |
| `closeModal()` | ~1694 | Закриває модалку **і скидає форму** |
| `initPortfolio()` | ~1533 | Ініціалізує crossfade-слайдер з Ken Burns |
| `pfGo(direction)` | ~1615 | Перегортає фото портфоліо; новий клік доклацує поточний кросфейд (`pfFinish`), не блокується |
| `pfSelectProject(index, btn)` | ~1623 | Перемикає проєкт в портфоліо |
| `getProjectImages(project)` | ~1501 | Повертає `imagesMobile` на мобільному, `images` на десктопі |
| `loadScriptOnce(src, name)` | ~1449 | Lazy-loader скриптів; дедуплікує по `globalName` |
| `setText(id, value)` | ~1521 | Безпечний setter `textContent` по id |
| `applyPhoneMask(input)` | ~1747 | Маска UA-телефону; ідемпотентна через `dataset.maskApplied` |

## Локальний запуск і тести

```bash
# Сервер
python3 -m http.server 8900 --directory /home/user/geometriya-sadu
# http://localhost:8900

# Playwright (headless Chromium)
node /tmp/your-check.js
# node: /opt/node22/bin/node
# playwright: /opt/node22/lib/node_modules/playwright
```

## Поточний PageSpeed (mobile)
Performance **93–95** · LCP 2.2с · TBT 100мс · CLS 0

## Незавершене / на майбутнє
- 15 невикористаних зображень у `images/` (запасні хіро: `Heroback*.webp`, `hero.webp`, `why-sketch*.webp` та ін.) — лишені як запас, можна прибрати за потреби

---

## 🔐 Безпека — зробити при переїзді на хост

При перенесенні сайту з GitHub Pages на продакшн-хостинг внести такі правки:

### 1. Web3Forms — обмежити домен (5 хв, критично)
У дашборді Web3Forms → **Settings → Allowed Domains** → додати лише домен продакшну (напр. `geometriya-sadu.com`).  
Без цього будь-хто може слати запити напряму з іншого домену.

### 2. Content Security Policy (CSP)
Додати `<meta>` в `<head>` або HTTP-заголовок від хостингу:
```html
<meta http-equiv="Content-Security-Policy"
  content="default-src 'self'; script-src 'self' 'unsafe-inline'; style-src 'self' 'unsafe-inline'; img-src 'self' data:; font-src 'self'; connect-src https://api.emailjs.com; frame-ancestors 'none';">
```
`frame-ancestors 'none'` також закриває clickjacking на формі.

### 3. SRI-хеші для vendored скриптів
Згенерувати та додати `integrity=` для `vendor/gsap.min.js`, `ScrollTrigger.min.js`, `email.min.js`, `lenis.min.js`:
```bash
# генерація хешу
openssl dgst -sha256 -binary vendor/gsap.min.js | openssl base64 -A
```
```html
<script src="vendor/gsap.min.js" integrity="sha256-XXXX" crossorigin="anonymous"></script>
```

### 4. Замінити `document.write` на безпечний варіант
Рядок ~11 в `index.html` — замінити на `createElement`:
```js
const preload = document.createElement('link');
preload.rel = 'preload';
preload.as = 'image';
preload.type = 'image/webp';
preload.href = 'images/' + (window.innerWidth <= 768 ? 'Backv5-m' : 'Backv5') + '.webp';
document.head.appendChild(preload);
```

### 5. EmailJS шаблон — перевірити екранування
У редакторі шаблону EmailJS переконатись, що всі змінні використовують `{{variable}}` (з екранув.), а не `{{{variable}}}` (без екранув.) — інакше XSS в листах.
