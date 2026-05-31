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
- Демо/превʼю (не в продакшені): `services-photo-preview.html`, `hover-preview.html` та ін.

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

## Поточний стан сайту (актуально на PR #188)

### Типографіка (глобально)
- **Навбар** `.gs-nav`: `13px / 700 / uppercase / 0.08em / color:#fff`
- **Кнопка в навбарі** `.gs-cta-pill`: `13px / 700 / uppercase / 0.08em / color:#fff`
- **Кнопки** `.gs-btn`: `13px / 700 / uppercase / 0.08em`
- Бургер: без рамки, лінії `#F28D1B`

### #hero
- Фон: `images/Heroback3.webp` (вечірня сцена з туями/самшитом)
- Кастомний курсор `#gs-dot` — помаранчева куля, `body{cursor:none}`
  - На `.gs-cta-orange` → **білий**
- Scramble + word-reveal анімація заголовків

### #why
- Фон: `images/backwhy.webp` з паралаксом GSAP (`yPercent:20`)

### #services — Картки послуг (`.gs-pcard`)
- Вертикальні photo-картки, `aspect-ratio:3/4`, сітка `repeat(3,1fr)`, `column-gap:35px; row-gap:56px`
- **Всі 8 карток мають реальні фото** (`.gs-pcard-img`)
- Структура картки: `<img class="gs-pcard-img">` → `.gs-pcard-overlay` → `.gs-pcard-cat` → `.gs-pcard-arrow` → `.gs-pcard-body` (`.gs-pcard-label`, `.gs-pcard-title`, `.gs-pcard-sub`)

#### Анімація появи (IntersectionObserver + клас `.is-in`)
- `.gs-pcard::before` — діагональна шторка (`skewY(8deg)`), keyframe `gsBgEnter` сповзає вниз 1.2s
- Заголовки — `gsTextUp`: `translateY(120%) skewY(6deg)` + `blur(8px)→0`, каскад label→title→sub
- **ВАЖЛИВО**: `gsBgZoomOut` на фото видалена — фото стартує з `scale(1)`, hover дає `scale(1.08)` одразу

#### Hover на картках
- Фото: `transform:scale(1.08)`, `transition:1.2s cubic-bezier(0.22,1,0.36,1)`
- Стрілка: зеленіє + glitch + поворот 180°

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

### #cta — Форма заявки (актуальний стан після PR #188)
- Секція: клас `.gs-cta-orange`, фон `#0c160e` + **меш-градієнт анімований**
  - `background-image`: 5 radial-gradient блобів (помаранчевий + зелений), `background-size:150-160%`
  - Анімація `gs-mesh-flow` 22s — плавне переливання кольорів через `background-position`
  - Шум `::before`: inline SVG fractalNoise, `opacity:0.30`, `mix-blend-mode:overlay`
  - Скетч SVG **прибраний** (`display:none`)
- Курсор `#gs-dot` → **білий** на `.gs-cta-orange`
- Форма-панель `.gs-oframe`:
  - `background: rgba(255,255,255,0.06)`, `border: 1px solid rgba(255,255,255,0.12)`
  - `backdrop-filter: blur(16px)`, `border-radius: 18px`
  - `box-shadow: 0 24px 60px -30px rgba(0,0,0,0.6), inset 0 1px 0 rgba(255,255,255,0.10)`
- Кутові дужки `.gs-oframe-corner`:
  - Колір **білий** `#fff`, радіус **18px** (як у форми)
  - Анімація **ззовні всередину** (від кутів до форми)
- Поля форми: `color:#fff`, `border-bottom: rgba(255,255,255,0.28)`, placeholder `#fff`
- Кнопка `.gs-osend`: `color:#fff`, стрілка `#fff` → зелена на ховер
- Нотатка `.gs-oform-note`: `color: rgba(255,255,255,0.38)`

### Інші секції
- **#process**: таби `.gs-chip` з text-scramble гліч-ефектом
- **FAB-віджет**: кнопка форми через `openModal()` (без аргументів!)
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
