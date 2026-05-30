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
- `geometriya-sadu-VANILLA.html` — **головний робочий файл** (~2000 рядків, всі правки тут)
- `index.html` — редірект на головний файл
- `.github/workflows/pages.yml` — деплой на GitHub Pages з гілки `main`
- Бекапи: `geometriya-sadu-VANILLA-backup-*.html`
- Демо/превʼю (не в продакшені): `services-photo-preview.html`, `cta-sketch-variants-preview.html` та ін.

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

### Робота з кількома сесіями Claude
Користувач іноді паралельно має дві активні сесії Claude. Щоб уникнути конфліктів:
- **Тільки одна сесія** вносить правки одночасно
- Перед кожною правкою обов'язково `git fetch origin main && git reset --hard origin/main`
- Після мерджу PR — інша сесія теж робить reset перед наступною зміною

---

## Поточний стан сайту (актуально на PR #145)

### #services — Картки послуг (`.gs-pcard`)
- Вертикальні photo-картки, `aspect-ratio:3/4`, сітка `repeat(3,1fr)`, `column-gap:35px; row-gap:56px`
- **Всі 8 карток мають реальні фото** (`.gs-pcard-img`)
- Структура картки: `<img class="gs-pcard-img">` → `.gs-pcard-overlay` → `.gs-pcard-cat` → `.gs-pcard-arrow` → `.gs-pcard-body` (`.gs-pcard-label`, `.gs-pcard-title`, `.gs-pcard-sub`)

#### Анімація появи (IntersectionObserver + клас `.is-in`)
- `.gs-pcard::before` — діагональна шторка (`skewY(8deg)`), keyframe `gsBgEnter` сповзає вниз 1.2s
- Фото — `gsBgZoomOut` 1.7s: `scale(1.1)→1` + `blur(10px)→0`
- Заголовки — `gsTextUp`: `translateY(120%) skewY(6deg)` + `blur(8px)→0`, каскад label→title→sub
- Каскад по колонках: `--reveal-delay: 0.1 + col*0.18s`

#### Hover на картках
- Фото: `transform:scale(1.08)`, `transition:1.2s cubic-bezier(0.22,1,0.36,1)` — плавний повільний zoom
- Стрілка: зеленіє + glitch (`gs-icon-glitch`) + поворот 180°
- Більше нічого (зелена шторка/reveal опису — відхилено)

#### Фото файли (`images/`)
| Картка | Файл |
|--------|------|
| Ландшафтне проєктування | `Ландшафтне проєктування.webp` |
| Озеленення та благоустрій | `Озеленення .webp` (є пробіл!) |
| Рулонний газон | `Рулонний газон v2.webp` |
| Земляні роботи | `Підготовчі роботи.webp` |
| Монтаж автополиву | `Автополив.webp` |
| Догляд за садом | `Сервісне обслуговування.webp` |
| Консервація автополиву | `Обслуговування автополиву.webp` |
| Корпоративним клієнтам | `Корпоративним клієнтам.webp` |

### #services — Анімація заголовків секцій (PR #143–#145)
- Заголовки `.h-display-2` та `.gs-cta-heading` розбиваються по словах через JS `splitWords()`
- Кожне слово: `.w-wrap` (overflow:hidden) → `.w` (анімується знизу зі скосом + blur)
- Стиль анімації аналогічний `.gs-pcard-title` (skew + blur rise)
- CSS класи: `.w-wrap`, `.w`, анімація через `gsTextUp` keyframe

### #cta — Форма заявки
- Секція: клас `.gs-cta-orange`, фон `#f4f3ef`, колір тексту `#0a0b0a`
- Фоновий скетч `images/cta-sketch.svg` — **прямий дочірній елемент `<section>`** (не у враппері!)
  - CSS: `position:absolute; left:50%; bottom:0; transform:translateX(-50%); width:min(1400px,118%); opacity:0.5`
  - Фільтр (зелений): `filter:invert(38%) sepia(40%) saturate(700%) hue-rotate(96deg) brightness(85%)`
  - `mix-blend-mode:multiply` — **КРИТИЧНО**: якщо обгорнути в div зі своїм z-index/position — з'явиться білий прямокутник
  - Анімація: `clip-path:inset(0 0 100% 0)` → `inset(0 0 0%)` (GSAP, 1.8s, scrollTrigger)
- Форма-панель `.gs-oframe`: `background:#fff; border-radius:14px` (суцільно біла)
- Кутові дужки `.gs-oframe-corner` — помаранчеві `#F28D1B`
- Нотатка про персональні дані `.gs-oform-note` — **всередині** `.gs-oframe` після `</form>`
- Кнопка `.gs-osend`: стрілка помаранчева → зелена на ховер + поворот 180°
- Курсор `#gs-dot` при наведенні на `.gs-cta-orange` стає зеленим (клас `.on-orange` через JS)

### Інші секції
- **#hero**: кастомний курсор-куля (помаранчева `#gs-dot`, `body{cursor:none}`), scramble + word-reveal анімація заголовків
- **#why**: фон `Back.webp` з паралаксом GSAP (yPercent:20)
- **#process**: таби `.gs-chip` з text-scramble гліч-ефектом
- **FAB-віджет**: зелені іконки, зациклена гліч-анімація, пульсація
- **#marquee**: іконки з гліч-ефектом (rotation + orange flash)
- **Таблиці цін**: 4 таби

---

## Незавершені задачі / на майбутнє
- **Telegram-лінк**: зараз `href="#"` — потрібне посилання від клієнта
- Реальні фото в секції «Кроки» і 3-му проєкті портфоліо
- Почистити старий CSS/JS класів `.gs-svc*` (лишився, не використовується)
- Видалити демо-файли після фіналізації

## Стиль роботи з користувачем
- Спілкування українською
- Користувач надсилає скріншоти + короткий опис → Claude вносить у HTML, комітить, мерджить
- Тексти мають звучати «від людини з досвідом у сфері», не від маркетолога
