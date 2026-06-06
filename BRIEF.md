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
- **Why** (`#why`) — горизонтальний слайдер на GSAP ScrollTrigger (canvas-крапки, без фото-фону)
- **Портфоліо** (`#portfolio`) — слайдер фонів, масиви `portfolioProjects[].images` / `.imagesMobile`
- **Ціни** (`#pricing`), **FAQ** (`#faq`), **Footer**

## Технічне
- Шрифт: **Onest** (локальний `fonts/`), один на весь сайт
- Анімації: **GSAP + ScrollTrigger** (`vendor/`) — підключені БЕЗ `defer` (defer ламає hero-анімації)
- Форма: **EmailJS** (lazy-load), honeypot анти-спам
- Git: автор комітів `Claude <noreply@anthropic.com>`, гілка розробки `claude/busy-cerf-uH3lP` → `main`

## Поточний PageSpeed (mobile)
Performance **93–95** · LCP 2.2с · TBT 100мс · CLS 0

## Незавершене / на майбутнє
- 15 невикористаних зображень у `images/` (запасні хіро: `Heroback*.webp`, `hero.webp`, `why-sketch*.webp` та ін.) — лишені як запас, можна прибрати за потреби
