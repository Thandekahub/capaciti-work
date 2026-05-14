# capaciti-work
projects and assignments for capaciti
Here's a detailed breakdown of how this app was built:

1. Foundation & Stack

- Framework: TanStack Start (React 19 + Vite 7) — a full-stack React framework with file-based routing and SSR
- Styling: Tailwind CSS v4 with semantic design tokens defined in `src/styles.css` (using `oklch` color space for a modern SaaS palette + custom gradients and shadow variables like `--shadow-elegant`)
- UI Components: shadcn/ui primitives (Card, Button, Sidebar, Textarea, etc.) for a consistent, accessible component library
- Icons: Lucide React for clean, professional iconography
- Backend: Lovable Cloud (managed Supabase) wired up for future persistence/auth
- AI: Lovable AI Gateway (Google Gemini `gemini-3-flash-preview`) — no API key setup needed

2. Architecture

Routing (file-based)
Each feature gets its own route file under `src/routes/`:
- `index.tsx` → Dashboard (cards linking to all tools)
- `email.tsx` → Smart Email Generator
- `meetings.tsx` → Meeting Notes Summarizer
- `tasks.tsx` → AI Task Planner
- `research.tsx` → Research Assistant
- `chat.tsx` → Conversational AI

The route tree is auto-generated into `routeTree.gen.ts`.

Layout
`src/routes/__root.tsx` wraps everything in:
- `QueryClientProvider` (TanStack Query for async state)
- `SidebarProvider` + `AppSidebar` (collapsible left navigation)
- A sticky top header with `SidebarTrigger`
- `<Outlet />` for the active route
- Global `Toaster` for notifications

3. AI Layer (`src/lib/ai.functions.ts`)

This is the core. Two TanStack server functions (`createServerFn`) act as type-safe RPCs that run only on the server:

`generateAI` — for the structured tools
- Takes `{ feature, input }` validated with Zod
- A `FEATURES` map holds structured prompts per tool. Each prompt:
  - Sets a clear role ("You are an executive assistant…")
  - Defines explicit output structure (Markdown sections like `## Summary`, `## Action Items`, `## Deadlines`)
  - Injects user inputs (tone, audience, notes, etc.)
- POSTs to `https://ai.gateway.lovable.dev/v1/chat/completions` with `LOVABLE_API_KEY` (read inside the handler from `process.env`)
- Handles 429 (rate limit) and 402 (out of credits) with friendly messages

`chatAI` — for conversational interface
- Accepts a `messages[]` array (role + content) for multi-turn context
- Uses an "Aura" system persona prompt

4. Shared Components

- `FeatureForm.tsx` — Reusable form wrapper handling submit state, validation, and triggering the server function
- `AIOutput.tsx` — Renders results with:
  - Loading skeleton/spinner
  - Error state
  - Copy-to-clipboard button
  - Built-in disclaimer: "AI-generated content may require human review"
- `PageHeader.tsx` — Consistent icon + title + description for each page
- `AppSidebar.tsx` — Navigation with active-route highlighting

5. Per-Feature UX

| Feature | Inputs | Output |
|---|---|---|
| Email | Audience, tone, purpose | Subject + body email |
| Meetings | Raw notes/transcript | Summary, key points, action items, deadlines |
| Tasks | Task list + context | Prioritized P1/P2/P3 plan with time blocks |
| Research | Topic + context | TL;DR, insights, risks, next steps |
| Chat | Free-form messages | Streaming-style threaded replies |

6. Design Principles Applied

- Semantic tokens only — no hardcoded colors in components; everything pulls from CSS variables for theme consistency
- Responsive — sidebar collapses on mobile, max-width content container, fluid padding
- Loading states — every AI call shows a bouncing-dot indicator
- Error surfacing — gateway errors (rate limit, credits) bubble up to user-visible toasts/messages
- Disclaimer — present in sidebar footer and on every AI output

Want me to dive deeper into any specific part — the prompt engineering, the design system tokens, or how the server functions work?
