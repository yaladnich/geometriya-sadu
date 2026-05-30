# Бриф для нового чату — «Геометрія саду»

> Передай цей файл на початку нового чату, щоб продовжити роботу без втрати контексту.

## Живий сайт
https://yaladnich.github.io/geometriya-sadu/

## Що це за проєкт
Односторінковий лендинг ландшафтної компанії **«Геометрія саду»** (Київ та область).
Послуги: проєктування, рулонний газон, автополив, озеленення, земляні роботи, догляд, корпоративним клієнтам.

## Стек
- Чистий **vanilla HTML/CSS/JS** в одному файлі
- **GSAP + ScrollTrigger** — скрол-анімації
- **Lenis** (`@studio-freight/lenis@1.0.42`) — плавний скрол
- **Tabler Icons webfont** (`@tabler/icons-webfont@3.19.0`)
- **EmailJS** — форма заявки
  - publicKey: `jo5qqxaBsOw0U6wKb`
  - service: `service_blkynfj`
  - template: `template_m57n5fv`
- Брендові іконки (Telegram/Viber/телефон/форма) — inline SVG

## Файли
- `geometriya-sadu-VANILLA.html` — **головний робочий файл** (~1900 рядків, всі правки тут)
- `index.html` — редірект на головний файл
- `.github/workflows/pages.yml` — деплой на GitHub Pages з гілки `main`
- Бекапи: `geometriya-sadu-VANILLA-backup-*.html`
- Демо/превʼю (не в продакшені): `services-photo-preview.html`, `services-video-preview.html`, `services-3d-preview.html`, `leaf-variants-preview.html`, `cta-sketch-variants-preview.html`

## Деплой
- **GitHub Pages** через GitHub Actions, тригер — пуш у `main`
- Сайт оновлюється ~1 хв після мерджу в main
- Контейнер НЕ має доступу до `github.io` (network policy) — деплой перевіряє користувач у браузері

## Git workflow (ВАЖЛИВО)
Репозиторій: **`github.com/yaladnich/geometriya-sadu`** (owner=`yaladnich`, repo=`geometriya-sadu`).
Прямий пуш у `main` заблоковано — працюємо через PR:
1. Робоча гілка: `claude/beautiful-cannon-AWQJW`
2. **Перед кожною правкою**: `git fetch origin main && git reset --hard origin/main` (бо squash-merge переписує історію — інакше будуть конфлікти)
3. Внести правку → `git commit` → `git push -f origin claude/beautiful-cannon-AWQJW`
4. PR через MCP (`mcp__github__create_pull_request`, base=main) → squash merge (`mcp__github__merge_pull_request`)
- Claude GitHub App встановлено на репо (write-доступ). MCP **не може** створювати гілки (403) — гілку створювати через `git push`.

---

## Поточний стан секцій

### #services — Картки послуг (`.gs-pcard`)
- Вертикальні photo-картки, `aspect-ratio:3/4`, сітка `repeat(3,1fr)`, `column-gap:35px; row-gap:56px`
- Зараз замість фото — **градієнтні плейсхолдери** `.gs-pcard-ph`
- Для підключення реального фото: замінити `<div class="gs-pcard-ph"></div>` на `<img class="gs-pcard-img" src="images/...">` (CSS для `.gs-pcard-img` вже є)
- Структура картки: `.gs-pcard-ph` → `.gs-pcard-overlay` → `.gs-pcard-cat` → `.gs-pcard-arrow` → `.gs-pcard-body` (`.gs-pcard-label`, `.gs-pcard-title`, `.gs-pcard-sub`)
- Анімація появи: `.gs-pcard::before` шторка (`gsBgEnter`), фото (`gsBgZoomOut`), заголовки (`gsTextUp`), тригер IntersectionObserver + клас `.is-in`
- Стрілка `.gs-pcard-arrow`: помаранчева за замовч., ховер → зеленіє + гліч + поворот 180°

#### Промпти Midjourney/ImageFX для карток (vertical portrait 3:4):
| # | Назва | Промпт |
|---|-------|--------|
| 01 | Проєктування | `architect reviewing garden design blueprints on tablet, scale model of modern garden layout on wooden table, pencil sketches, soft natural light, dark moody studio, vertical portrait 3:4` |
| 02 | Рулонний газон | `workers laying fresh rolled sod lawn at modern suburban house, lush green grass rolls, early morning light, mist, cinematic, dark moody, vertical portrait 3:4` |
| 03 | Автополив | `underground irrigation system installation, spray heads watering lush green lawn at dusk, water droplets in golden light, cinematic, dark moody, vertical portrait 3:4` |
| 04 | Озеленення | `planting ornamental shrubs and perennials in modern garden, mulched beds, stone path, contemporary house facade, golden hour, cinematic, vertical portrait 3:4` |
| 05 | Земляні роботи | `mini excavator doing precision landscaping earthworks, fresh soil, modern suburban garden construction, dramatic overcast sky, cinematic, dark moody, vertical portrait 3:4` |
| 06 | Корпоративним клієнтам | `manicured corporate office building entrance garden, perfectly trimmed hedges, seasonal flowers, glass facade, morning light, cinematic, vertical portrait 3:4` |
| 07 | Декоративні елементи | `decorative garden water feature with stone boulders, ornamental grasses, modern garden design, evening moody light, cinematic, dark atmosphere, vertical portrait 3:4` |
| 08 | Сервісне обслуговування | `professional gardener trimming geometric hedges with electric hedge trimmer, lawn aeration and mowing in background, modern suburban garden, morning dew, golden hour, cinematic, vertical portrait 3:4` |

### #cta — Форма заявки
- Секція: клас `.gs-cta-orange`, фон `#f4f3ef`, колір тексту `#0a0b0a`
- Фоновий скетч `images/cta-sketch.svg` — прямий дочірній елемент `<section>`, зелений фільтр + `mix-blend-mode:multiply`
  - CSS: `position:absolute; left:50%; bottom:0; transform:translateX(-50%); width:min(1400px,118%); opacity:0.5`
  - Фільтр: `filter:invert(38%) sepia(40%) saturate(700%) hue-rotate(96deg) brightness(85%)`
  - Анімація: `clip-path:inset(0 0 100% 0)` → `inset(0 0 0%)` (GSAP, 1.8s, scrollTrigger)
  - **ВАЖЛИВО**: скетч має бути прямим дочірнім елементом `<section>` — не у враппері div, бо проміжний stacking context ламає `mix-blend-mode:multiply` (білий фон стає видимим)
- Форма-панель `.gs-oframe`: `background:#fff; border-radius:14px; box-shadow:...` (суцільно біла, без прозорості)
- Кутові дужки `.gs-oframe-corner` — помаранчеві `#F28D1B`
- Нотатка про персональні дані — **всередині** `.gs-oframe` після `</form>` (клас `.gs-oform-note`)
- Кнопка `.gs-osend`: стрілка помаранчева → зелена на ховер + поворот 180°
- Курсор `#gs-dot` при наведенні на `.gs-cta-orange` стає зеленим (клас `.on-orange` через JS mouseover)

### Інші секції (зроблено раніше)
- **#hero**: кастомний курсор-куля (помаранчева `#gs-dot`, `body{cursor:none}`), scramble-анімація кнопок
- **#why**: фон `Back.webp` з паралаксом GSAP (yPercent:20)
- **#process**: таби `.gs-chip` з text-scramble гліч-ефектом
- **FAB-віджет**: зелені іконки, зациклена гліч-анімація, пульсація
- **#marquee**: іконки з гліч-ефектом (rotation + orange flash)
- **Таблиці цін**: 4 таби

---

## Незавершені задачі / на майбутнє
- **Реальні фото в картки послуг** — зараз плейсхолдери (пріоритет! промпти вище)
- **Telegram-лінк**: зараз `href="#"` — потрібне посилання від клієнта
- Реальні фото в секції «Кроки» і 3-му проєкті портфоліо
- Почистити старий CSS/JS класів `.gs-svc*` (лишився, не використовується)
- Видалити демо-файли після фіналізації

## Стиль роботи з користувачем
- Спілкування українською
- Користувач надсилає скріншоти + короткий опис → Claude вносить у HTML, комітить, мерджить
- Тексти мають звучати «від людини з досвідом у сфері», не від маркетолога
