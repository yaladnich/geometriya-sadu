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
- Демо/превʼю (не в продакшені): `services-photo-preview.html`, `services-video-preview.html`, `services-3d-preview.html`, `leaf-variants-preview.html`

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

## Остання сесія — РЕДИЗАЙН КАРТОК ПОСЛУГ (стиль lev-development.com.ua)
Секцію `#services` повністю переробили під стиль lev-development.com.ua.

### Структура картки (клас `.gs-pcard`)
- Вертикальні photo-картки, `aspect-ratio:3/4`, сітка `repeat(3,1fr)`, `column-gap:35px; row-gap:56px`
- Зараз замість фото — **градієнтні плейсхолдери** `.gs-pcard-ph` (linear-gradient accent-deep→bg-1)
- Елементи: `.gs-pcard-ph` → `.gs-pcard-overlay` → `.gs-pcard-cat` (вертикальний лейбл) → `.gs-pcard-arrow` → `.gs-pcard-body` (`.gs-pcard-label`, `.gs-pcard-title`, `.gs-pcard-sub`)
- 8 карток, всі звичайні (`.is-soon` лишився в CSS, але не використовується)
- Для підключення реального фото: замінити `<div class="gs-pcard-ph"></div>` на `<img class="gs-pcard-img" src="images/...">` (CSS для `.gs-pcard-img` вже є)

### Анімація появи (точно з main.css lev-development)
- `.gs-pcard::before` — діагональна шторка (`skewY(8deg)`), keyframe **`gsBgEnter`** сповзає вниз 1.2s
- Фото — **`gsBgZoomOut`** 1.7s: `scale(1.1)→1` + `blur(10px)→0`
- Заголовки — **`gsTextUp`**: `translateY(120%) skewY(6deg)` + `blur(8px)→0`, каскад label→title→sub
- Тригер: **IntersectionObserver** додає клас `.is-in`; каскад по колонках через `--reveal-delay` (`0.1 + col*0.18s`)

### Стрілка `.gs-pcard-arrow`
- Помаранчева `rgba(242,141,27,.78)` за замовчуванням
- Ховер: зеленіє + **glitch** (`gs-icon-glitch` через JS mouseenter) + **поворот 180°**
- JS-loop: `document.querySelectorAll('.gs-svc, .gs-pcard')`

### PR цієї сесії (всі в main)
#83 редизайн · #84 картки 07/08 + анімація заголовків · #85 blur · #86 гліч-стрілка · #87 поворот 180°

## Раніше зроблено
- Консолідація HTML, GitHub Pages, мобільна адаптація (бургер ≤900px)
- SEO-переписування H2, таблиці цін (4 таби)
- FAB-віджет контактів (згортання, каскад, пульсація)
- Блок Why (#why): фон `Back.webp` з паралаксом (GSAP ScrollTrigger, yPercent:20)
- Зелений скролбар (#257c4d)
- Таби (.gs-chip): text-scramble гліч-ефект

## Незавершені задачі / на майбутнє
- **Реальні фото в картки послуг** — зараз плейсхолдери (пріоритет!)
- **Telegram-лінк**: зараз `href="#"` — користувач ще не дав посилання
- Реальні фото в секції «Кроки» і 3-му проєкті портфоліо
- Почистити старий CSS/JS класів `.gs-svc*` (лишився, не використовується)
- Видалити демо-файли після фіналізації

## Стиль роботи з користувачем
- Спілкування українською
- Користувач надсилає скріншоти + короткий опис → Claude вносить у HTML, комітить, мерджить
- Тексти мають звучати «від людини з досвідом у сфері», не від маркетолога
