# Richard Flamenco — Plan Technique d'Implémentation

## Context
Richard is a flamenco guitar teacher in Montpellier, France. His current WordPress site (richard-flamenco.com) is outdated with broken pages, no media showcase, and poor mobile experience. The goal is to build a modern, fast, bilingual (FR/EN) site that Richard can manage entirely from his iPhone via Airtable.

## Tech Stack
| Layer | Tool |
|-------|------|
| Frontend | **Astro 5** + **Tailwind CSS 3** |
| CMS/Data | **Airtable** (base: `appjFKfl9TowmzXym`) |
| Hosting | **Netlify** (static + functions) |
| Code | **GitHub** (`netventureFrance/Richard-Flamenco`) |
| Videos | **YouTube** embeds (privacy-enhanced) |
| Booking | **Cal.com** (free, embeddable) |
| Email | **Resend** (contact form notifications) |
| Images | **Cloudinary** (URL-based transforms) |
| Analytics | **Plausible** (GDPR-friendly, no cookie banner) |
| i18n | Astro built-in — `/fr/` and `/en/` prefixes |
| Instagram | Integration on site |

## Airtable Schema (7 Tables)

**Courses** — Name_FR, Name_EN, Slug, Description_FR, Description_EN, Level (select), Type (select), Image_URL, Duration, Schedule_FR, Schedule_EN, Active (checkbox), Sort_Order

**Pricing** — Formula_Name, Level (select), Price (currency), Price_Type (select), Includes_FR, Includes_EN, Individual_Hours, Workshop_Hours, Popular (checkbox), Active, Sort_Order

**Concerts** — Title_FR, Title_EN, Slug, Date, Time, Venue, City, Address, Description_FR, Description_EN, Poster_URL, Performers, Ticket_URL, Status (select), Published

**Compositions** — Title, Slug, YouTube_URL, YouTube_ID, Description_FR, Description_EN, Style (select), Featured (checkbox), Published, Sort_Order

**Blog** — Title_FR, Title_EN, Slug, Body_FR (markdown), Body_EN (markdown), Excerpt_FR, Excerpt_EN, Cover_Image_URL, Category (select), Published_Date, Published

**Testimonials** — Student_Name, Quote_FR, Quote_EN, Photo_URL, Rating, Course_Type, Published, Sort_Order

**Site Settings** (single row) — Site_Title_FR/EN, Phone, Email, Address, WhatsApp_Number, Instagram_URL, Facebook_URL, YouTube_URL, Bio_FR/EN, Hero_Image_URL, Hero_Title_FR/EN, Hero_Subtitle_FR/EN, CalCom_Username, Google_Maps_Embed_URL

## Project Structure
```
richard-flamenco/
├── astro.config.mjs
├── tailwind.config.mjs
├── netlify.toml
├── package.json
├── .env.example
├── public/
│   ├── favicon.svg
│   └── robots.txt
├── src/
│   ├── content/config.ts          # Airtable content collections
│   ├── i18n/
│   │   ├── ui.ts                  # UI translations (nav, buttons, labels)
│   │   ├── utils.ts               # getLangFromUrl(), useTranslations(), localized()
│   │   └── routes.ts              # FR/EN route mapping for language switcher
│   ├── layouts/
│   │   ├── BaseLayout.astro       # HTML head, fonts, analytics
│   │   └── PageLayout.astro       # BaseLayout + Header + Footer
│   ├── components/
│   │   ├── common/                # Header, Footer, LanguageSwitcher, MobileMenu, SEOHead
│   │   ├── home/                  # Hero, CoursePreview, TestimonialSlider, YouTubeFeature
│   │   ├── courses/               # CourseCard, LevelBadge
│   │   ├── pricing/               # PricingCard, PricingTable
│   │   ├── concerts/              # EventCard, EventTimeline
│   │   ├── media/                 # YouTubeEmbed, VideoGallery
│   │   ├── blog/                  # BlogCard, BlogContent
│   │   └── contact/               # ContactForm, MapEmbed, CalBooking, WhatsAppButton
│   ├── pages/
│   │   ├── index.astro            # Redirect → /fr/
│   │   ├── fr/                    # French pages
│   │   └── en/                    # English pages
│   ├── utils/
│   │   ├── cloudinary.ts          # Image URL transforms
│   │   └── formatting.ts          # Date/price formatting
│   └── styles/global.css          # Tailwind directives + component classes
└── netlify/functions/
    └── submission-created.mts     # Resend email on form submit
```

## Design
- **Colors**: Flamenco palette — warm amber/orange (`flamenco-500: #ee7712`), deep red (`rojo-500: #dc2626`), near-black (`negro-950: #1a1a1a`), gold accent (`oro-400: #fbbf24`)
- **Fonts**: Playfair Display (headings), Inter (body)
- **Style**: Dark header/footer, light page backgrounds, cards with subtle shadows, red CTA buttons
- **Mobile-first**: Single column → multi-column at md/lg breakpoints

## Pages
| FR Route | EN Route | Content |
|----------|----------|---------|
| `/fr/` | `/en/` | Landing: Hero → Courses → Bio → Video → Testimonials → CTA |
| `/fr/cours` | `/en/courses` | Course cards grouped by type |
| `/fr/tarifs` | `/en/pricing` | Pricing formula cards by level |
| `/fr/concerts` | `/en/concerts` | Upcoming + past events |
| `/fr/compositions` | `/en/compositions` | YouTube video gallery with style filter |
| `/fr/blog` | `/en/blog` | Blog post listing |
| `/fr/blog/[slug]` | `/en/blog/[slug]` | Individual blog post |
| `/fr/contact` | `/en/contact` | Form + Map + Cal.com booking + WhatsApp |
| `/fr/mentions-legales` | `/en/legal` | GDPR/legal |

## Content Pipeline (Richard's iPhone Workflow)
1. Richard edits a record in Airtable on his iPhone
2. Airtable Automation sends POST to Netlify Build Hook
3. Netlify rebuilds the static site (~2-3 min)
4. Site is live with updated content

## Implementation Phases

### Phase 1: Foundation
- Init Astro project, Tailwind, Netlify adapter, i18n config
- BaseLayout, PageLayout, Header (with mobile menu), Footer, LanguageSwitcher
- netlify.toml with redirects (root → /fr/, old WordPress URLs)
- Git init, push to `netventureFrance/Richard-Flamenco`, connect to Netlify

### Phase 2: Airtable Integration
- Install `@ascorbic/airtable-loader`
- `/src/content/config.ts` with all 7 collections
- i18n system: `ui.ts`, `utils.ts`, `routes.ts`
- Verify data fetching works with dev server

### Phase 3: Core Pages
- Courses page with CourseCard, LevelBadge
- Pricing page with PricingCard, PricingTable
- Compositions page with YouTubeEmbed (facade pattern), VideoGallery

### Phase 4: Home Page
- Hero with CTA
- CoursePreview cards
- Bio/About section
- Featured video
- TestimonialSlider (CSS scroll-snap)
- CTA banner

### Phase 5: Blog
- BlogCard, BlogContent components
- Blog listing + `[slug]` dynamic pages
- Markdown rendering with @tailwindcss/typography

### Phase 6: Contact & Events
- ContactForm (Netlify Forms + honeypot)
- Resend email notification function
- MapEmbed, CalBooking embed, WhatsAppButton (floating)
- Concerts page with EventCard

### Phase 7: Polish
- SEOHead (JSON-LD, OG tags, hreflang)
- Plausible analytics
- Instagram integration
- Legal pages
- Sitemap (astro-sitemap)
- Performance optimization, Lighthouse audit

### Phase 8: Launch
- Migrate real content to Airtable
- Custom domain setup
- Airtable Automations for auto-rebuild
- WordPress URL redirects for SEO continuity

## Verification
- `npm run dev` — local development
- `npm run build` — verify static build succeeds
- `astro check` — TypeScript validation
- Lighthouse audit (target: >90 performance)
- Test all pages in FR and EN on mobile + desktop
- Test contact form → Netlify submission → Resend email
- Test Airtable edit → webhook → Netlify rebuild
- Test WhatsApp link, Cal.com booking, YouTube embeds
- Verify old WordPress URLs redirect properly
