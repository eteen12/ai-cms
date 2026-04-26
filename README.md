# AI-Powered Blog CMS

![Next.js](https://img.shields.io/badge/Next.js_15-000000?style=for-the-badge&logo=nextdotjs&logoColor=white)
![React](https://img.shields.io/badge/React_19-20232A?style=for-the-badge&logo=react&logoColor=61DAFB)
![Supabase](https://img.shields.io/badge/Supabase-3ECF8E?style=for-the-badge&logo=supabase&logoColor=white)
![TailwindCSS](https://img.shields.io/badge/Tailwind_CSS-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)
![Claude](https://img.shields.io/badge/Claude_Haiku-D97757?style=for-the-badge&logo=anthropic&logoColor=white)
![OpenRouter](https://img.shields.io/badge/OpenRouter-6B4FBB?style=for-the-badge&logo=openai&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=for-the-badge&logo=javascript&logoColor=black)
![Vercel](https://img.shields.io/badge/Vercel-000000?style=for-the-badge&logo=vercel&logoColor=white)

> A production-grade, AI-assisted blog CMS built for small businesses that need to publish content that actually ranks — without hiring an SEO agency.

---

<!-- SCREENSHOT: Dashboard / Blog Hub overview -->
<!-- ![Dashboard Preview](./docs/images/dashboard.png) -->

---

## What Is This?

This is a fully custom, self-hosted blog content management system with a built-in AI writing assistant. It was built for a real landscaping business in Kelowna, BC — designed to let a non-technical business owner publish SEO-optimized blog posts without needing to understand SEO, schema markup, or meta tags at all.

The AI assistant doesn't just suggest edits in a chat window — it can **directly manipulate the post**, highlight specific passages, score the content on a 100-point rubric, and walk the author through exactly what needs to change and why. Every AI mutation is gated behind a confirmation step: you see exactly what will change before it happens.

This isn't a wrapper around WordPress. It's a lean, opinionated CMS built from scratch with a single goal: help a small business publish content that competes in local search and AI-powered results.

---

## Who It's For

**Small and medium business owners** who:

- Want to run a company blog but don't have a content team
- Are tired of fighting WordPress plugins and theme bloat
- Need blog content that ranks in Google *and* gets cited in AI Overviews (ChatGPT, Perplexity, Google AIO)
- Want full control over their own data without paying a SaaS CMS subscription forever

**Developers** building CMS solutions for clients who want:

- A clean, maintainable codebase they can actually own
- AI writing assistance that's grounded in real business context — not generic GPT output
- A modern stack (Next.js 15 App Router, Supabase, Tiptap v3) that won't be obsolete in two years

---

## Why It Matters for Businesses

Most small business websites have a blog that hasn't been touched in 18 months. The reason is always the same: writing SEO-optimized content is hard, slow, and expensive when outsourced.

This CMS solves that by embedding the expertise directly into the editor. The AI assistant is trained on:

- The specific business's services, URLs, and brand voice
- Google's E-E-A-T quality guidelines
- Local SEO signals (service area, climate zone, regional terminology)
- AI citation readiness — how to write content that gets quoted by Perplexity and Google AIO

A business owner opens the editor, writes a rough draft, clicks "Score this post," and gets a 100-point breakdown with specific, actionable feedback. The AI can fix the meta description, rewrite the title, add an FAQ section, or suggest better keywords — all with one click, all reviewed before committing.

The result: **publishable, competitive blog content in under an hour**, from someone who has never heard of E-E-A-T.

---

## Features

### Rich Text Editor
- **Tiptap v3** WYSIWYG editor with full formatting support
- Drag-and-drop image uploads directly into the post body
- Autosave every 30 seconds — no lost work
- Live character counters on meta title (60 char) and description (160 char)
- Preview mode before publishing

### AI Writing Assistant
- Slide-in assistant panel powered by **Claude Haiku** via OpenRouter
- **7 function tools** the AI can call:
  - `highlight_field` — pulses a CSS ring on any metadata field to draw attention
  - `highlight_editor_text` — highlights an exact passage inside the editor (non-destructive, clears after 5s)
  - `score_post` — renders a full 5-category score card in the chat
  - `update_field` — proposes a new value for any metadata field
  - `set_keywords` — proposes a new keyword array
  - `insert_content_at_cursor` — inserts HTML at the cursor
  - `replace_editor_content` — full document replacement (requires explicit confirmation)
- **Mutating tools are gated**: every content change shows a before/after diff card — Accept or Reject, one at a time
- **Streaming SSE response** — the assistant thinks and types in real time
- Quick-prompt buttons: "Score this post", "Audit my SEO", "Fix my meta description", and more

<!-- SCREENSHOT: AI Assistant panel open with a score card visible -->
<!-- ![AI Assistant](./docs/images/assistant-panel.png) -->

### SEO Out of the Box
- Full **Schema.org JSON-LD** on every post (`BlogPosting`, `HowTo`, `FAQPage` — set per post)
- **Open Graph + Twitter Card** metadata auto-populated from post fields
- Canonical URLs on all pages
- Auto-generated XML sitemap on every build via `next-sitemap`
- `robots.txt` with crawler-specific rules (blocks aggressive scrapers like Bytespider)
- `llms.txt` at the webroot for AI crawler context
- **Breadcrumb schema** on every individual post

### Post Management
- Draft / Published workflow with status badges
- Blog Hub dashboard with post stats at a glance
- Delete (drafts only), Edit, Preview actions per post
- Slug, category, author, publish date all editable

### Auth & Security
- Simple `httpOnly` cookie-based session (8-hour TTL)
- `ADMIN_PASSWORD` environment variable — no user table needed
- Supabase **Row Level Security** enforces public read = published only
- Server-side service role key for admin writes, anon key for public reads
- Security headers: HSTS, X-Frame-Options, X-Content-Type-Options, Referrer-Policy

---

## Architecture

```
Browser
  ├── Public Site (SSR/SSG)      → Supabase (anon key, RLS: published only)
  └── /admin (cookie-gated)
        ├── EditorForm.jsx        → Server Actions (save, upload, delete)
        │     └── Tiptap Editor   → Supabase Storage (blog-images bucket)
        └── AssistantPanel.jsx    → POST /api/admin/assistant (SSE stream)
                                       └── OpenRouter → Claude Haiku
```

```
/api/admin/assistant
  1. Verify admin_auth cookie
  2. Build system prompt:
       website.md          ← business facts, services, URLs, brand voice
       seo-principles.md   ← E-E-A-T, meta rules, local SEO, AI citations
       blog-writing.md     ← writing pillars, scoring rubric, anti-patterns
       output-protocol.md  ← tool-use rules, response format
       + live post snapshot (title, slug, HTML, plain text, word count, all fields)
  3. Stream response via SSE
  4. Accumulate tool_calls across chunks → emit tool_call events
  5. Client executes non-mutating tools immediately
  6. Client queues mutating tools → shows diff card → user accepts/rejects
  7. Tool results returned to model → stream continues
```

<!-- DIAGRAM: Architecture diagram image -->
<!-- ![Architecture](./docs/images/architecture.png) -->

---

## Database Schema

Single `posts` table in Supabase Postgres:

| Column | Type | Description |
|---|---|---|
| `id` | UUID | Primary key |
| `title` | TEXT | Post H1 |
| `slug` | TEXT | URL segment (`/blog/[slug]`) |
| `content` | TEXT | Tiptap HTML |
| `meta_title` | TEXT | `<title>` tag (60 char target) |
| `meta_description` | TEXT | Meta description (160 char target) |
| `keywords` | TEXT[] | Target keyword array (5–10) |
| `og_image_url` | TEXT | Hero image CDN URL |
| `og_image_alt` | TEXT | Alt text for accessibility + SEO |
| `category` | TEXT | Post category |
| `author` | TEXT | Byline |
| `schema_type` | TEXT | `BlogPosting` / `HowTo` / `FAQPage` |
| `status` | TEXT | `draft` or `published` (CHECK constraint) |
| `date_published` | DATE | ISO publish date |
| `date_modified` | DATE | Auto-updated on save |

**RLS policy**: `SELECT` is restricted to `status = 'published'` for the anon key. Admin operations use the service role key server-side only.

---

## Content Scoring Rubric

The AI's `score_post` tool evaluates every post on a 100-point scale:

| Category | Points |
|---|---|
| Content Quality | 30 |
| SEO Optimization | 25 |
| E-E-A-T Signals | 15 |
| Technical Elements | 15 |
| AI Citation Readiness | 15 |

**Score bands:**
- 90–100: Exceptional
- 80–89: Strong
- 70–79: Acceptable
- 60–69: Below Standard
- < 60: Rewrite Required

<!-- SCREENSHOT: Score card rendered in the assistant panel -->
<!-- ![Score Card](./docs/images/score-card.png) -->

---

## Tech Stack

| Layer | Technology |
|---|---|
| Framework | Next.js 15 (App Router) |
| UI | React 19, Tailwind CSS 3 |
| Editor | Tiptap v3 + custom ProseMirror extension |
| Database | Supabase (Postgres + Row Level Security) |
| File Storage | Supabase Storage |
| AI Model | Claude Haiku 4.5 via OpenRouter |
| AI Client | OpenAI SDK (OpenRouter-compatible) |
| Auth | httpOnly cookie + Next.js Server Actions |
| SEO | next-sitemap, Schema.org JSON-LD |
| Deployment | Vercel |

---

## Environment Variables

```env
# OpenRouter (AI)
OPENROUTER_API_KEY=

# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Admin
ADMIN_PASSWORD=
```

---

## Getting Started

```bash
# Install dependencies
npm install

# Set up environment variables
cp .env.example .env.local
# Fill in your Supabase project URL/keys, OpenRouter API key, and admin password

# Run the Supabase migration
# Apply /supabase/migrations/001_create_posts.sql and 002_add_image_and_schema_fields.sql
# via the Supabase dashboard SQL editor or the Supabase CLI

# Start the dev server
npm run dev
```

The CMS is available at `http://localhost:3000/admin`.

---

## Notable Technical Decisions

**Custom ProseMirror highlight extension** — The AI can highlight any passage inside the editor without touching the document. The extension builds a flat character index mapping every character position to its ProseMirror document position, enabling substring matching across nodes with mixed formatting (bold, italic, links). Falls back to whitespace-collapsed matching for robustness. Decorations clear automatically after 5 seconds.

**Mutating / non-mutating tool split** — The assistant can call tools freely, but only non-mutating tools (highlight, scroll, score) execute immediately. Any tool that would change the document or a field is queued in a `pendingConfirm` state. The user sees a diff card showing the current value and the proposed value and must accept or reject each one before the conversation continues. The LLM never writes directly to the database.

**Prompt structure for cache efficiency** — The system prompt is structured so the four static knowledge files form a stable prefix and only the live post snapshot (which changes every turn) is appended at the end. This maximizes cache hit rates on OpenRouter's prompt caching layer, reducing latency and cost on follow-up turns in the same session.

**Lazy Supabase client via Proxy** — Both Supabase clients (server and public) are wrapped in a `Proxy` object so `createClient()` is deferred until a property is first accessed. This prevents build-time crashes from missing environment variables during CI and local development without `.env.local`.

**Knowledge-grounded AI, not generic GPT** — The assistant's system prompt is built from four Markdown knowledge files committed to the repo. These files contain the specific business's services, URLs, brand voice, local SEO signals, and hard rules (no hallucinated statistics, no generic filler phrases). The AI's output reflects the actual business, not a generic landscaping company.

---

## Project Structure

```
/app
  /admin                  ← CMS (login, hub, editor, preview)
  /api/admin/assistant    ← SSE streaming AI endpoint
  /blog                   ← Public blog (index + [slug])
/components
  /admin                  ← EditorForm, AssistantPanel, DeletePostButton
  /layout                 ← Navbar, Footer
/lib
  /assistant
    tools.js              ← 7 AI tool schemas + MUTATING_TOOL_NAMES
    systemPrompt.js       ← Prompt builder (knowledge files + post snapshot)
    clientTools.js        ← Client-side tool executor + diff preview
    aiHighlightExtension.js ← Custom Tiptap/ProseMirror extension
    /knowledge            ← website.md, seo-principles.md, blog-writing.md, output-protocol.md
  deepseek.js             ← OpenAI SDK → OpenRouter
  supabase.js             ← Lazy-proxy Supabase clients
/supabase/migrations      ← SQL schema + RLS policies
```

---

## License

MIT

---

Built by [Ethan Breitkreutz](https://github.com/eteen12)
