# re-fix.kz — Сайт услуг ремонта корпусной мебели

## 1. Concept & Vision

Современный, профессиональный сайт для компании по ремонту корпусной мебели и бытовых услуг в Алматы. Дизайн должен вызывать доверие, демонстрировать мастерство и акцентировать внимание на качестве работ. Ощущение: надёжный мастер, который знает своё дело — чисто, аккуратно, профессионально.

## 2. Design Language

### Aesthetic Direction
Индустриальный минимализм с тёплыми акцентами. Чистые линии, много воздуха, фотографии работ как hero-элементы. Референс: скандинавский функциональный дизайн.

### Color Palette
- Primary: `#2C3E50` (тёмно-синий — надёжность)
- Secondary: `#E67E22` (тёплый оранжевый — акцент, CTA)
- Background: `#FAFAFA` (светло-серый)
- Surface: `#FFFFFF` (белый)
- Text Primary: `#2C3E50`
- Text Secondary: `#7F8C8D`
- Success: `#27AE60`

### Typography
- Headings: `Montserrat` (700, 600) — геометричный, современный
- Body: `Open Sans` (400, 500) — читаемый, нейтральный
- Fallback: system-ui, sans-serif

### Spatial System
- Base unit: 8px
- Section padding: 80px vertical
- Container max-width: 1200px
- Card gap: 24px
- Border radius: 8px (cards), 4px (buttons)

### Motion Philosophy
- Subtle entrance animations on scroll (fade-up, 400ms ease-out)
- Hover transitions: 200ms ease
- Smooth scroll between sections
- Button hover: slight lift with shadow

### Visual Assets
- Icons: Lucide Icons (CDN)
- Images: Unsplash (furniture, tools, interior)
- Decorative: тонкие линии-разделители, геометрические формы в SVG

## 3. Layout & Structure

### Single Page Structure
1. **Header** — Фиксированный, логотип слева, навигация справа, телефон prominent
2. **Hero** — Большое изображение, заголовок, подзаголовок, CTA кнопка
3. **Services** — Сетка услуг с иконками и описанием
4. **About/Why Us** — 3-4 преимущества с визуальным оформлением
5. **Portfolio** — Галерея выполненных работ (до/после концепция)
6. **Service Areas** — Карта покрытия (Алматы + 100км)
7. **CTA Section** — Форма обратной связи
8. **Footer** — Контакты, соцсети, копирайт

### Responsive Strategy
- Desktop: 1200px+ (полная сетка)
- Tablet: 768px-1199px (2 колонки)
- Mobile: <768px (1 колонка, hamburger menu)

## 4. Features & Interactions

### Navigation
- Smooth scroll to sections
- Active state highlighting on scroll
- Mobile: hamburger menu с slide-in панелью

### Service Cards
- Hover: lift + shadow + icon color change
- Click: scroll to contact form with service pre-selected

### Contact Form
- Fields: Имя, Телефон (required), Тип услуги (dropdown), Сообщение
- Validation: телефон формат Казахстана
- Submit: показ spinner → success message
- Error state: красная рамка + сообщение

### WhatsApp Button
- Floating button внизу справа
- Click: открывает WhatsApp с предзаполненным сообщением

## 5. Component Inventory

### Header
- Logo: "re-fix" + ".kz" (accent color)
- Nav links: Услуги, О нас, Портфолио, Контакты
- Phone: +7 (777) 123-45-67 (click-to-call)
- States: default, scrolled (add shadow + background blur)

### Hero Section
- Background: image с overlay gradient
- H1: "Ремонт корпусной мебели в Алматы"
- Subtitle: услуги + зона покрытия
- CTA: "Заказать замер" (primary button)

### Service Card
- Icon (40px, accent color)
- Title (Montserrat 600)
- Description (2-3 строки)
- States: default, hover (lift + shadow)

### Contact Form
- Input fields с labels
- Select dropdown
- Textarea
- Submit button (primary)
- States: default, focus, error, success

### Floating WhatsApp
- Fixed position bottom-right
- Green (#25D366) background
- WhatsApp icon
- Hover: scale 1.1

## 6. Technical Approach

### Stack
- Pure HTML5 + CSS3 + Vanilla JavaScript
- No frameworks — maximum performance
- Google Fonts CDN
- Lucide Icons CDN

### File Structure
```
/refix-kz
  index.html    — весь контент (HTML + embedded CSS + JS)
  SPEC.md      — спецификация
```

### Key Implementation Details
- CSS: Custom properties для цветов, Grid + Flexbox для layout
- JS: Intersection Observer для scroll animations, form validation
- Images: Unsplash URLs (optimized size parameters)
- SEO: Семантический HTML, meta tags, alt texts
