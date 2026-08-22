# Criar - AI Website Builder

Criar turns a natural-language website brief into a complete static website. Users choose a category, describe content, visual style, and functionality, then generate HTML, CSS, and JavaScript with Google Gemini. The result can be previewed, copied, exported, and deployed to Netlify.

The app also includes Clerk authentication, Prisma/PostgreSQL persistence, project history, contact and partnership forms, Stripe credit packages, and an AI prompt assistant.

## Problem It Solves

Building a first-draft website normally means either hiring a developer, wrestling with a drag-and-drop builder that fights you on custom design, or hand-writing HTML/CSS/JS from a blank file. Criar collapses that gap: a plain-language brief goes in, and a working, styled, responsive site comes out in seconds — previewable immediately, editable through further prompts or a drag-and-drop component panel, and deployable to a live URL without ever leaving the browser. It's aimed at the moment before a "real" build: prototyping an idea, mocking up a client concept, or shipping a small site fast.

## Source Code Access

This repository is private ([github.com/Laksh-Shetty/MPR_TM](https://github.com/Laksh-Shetty/MPR_TM)). To request collaborator access, reach out with your GitHub username via:

- **LinkedIn**: [linkedin.com/in/laksh-shetty-4bb576307](https://www.linkedin.com/in/laksh-shetty-4bb576307/)
- **Email**: [lakshshetty206@gmail.com](mailto:lakshshetty206@gmail.com)

## Screenshots

**App workspace** — the `/x` generation interface, with the refinement sidebar, live preview, and quick-edit shortcuts:

![Criar workspace](docs/screenshots/app-workspace.png)

**Example generated output** — an event landing page generated from a prompt, shown in the live preview:

![Generated hero section](docs/screenshots/example-hero.png)

![Generated content sections](docs/screenshots/example-sections.png)

## Demo

Live demo link: _coming soon_

## Features

- AI website generation and refinement with Gemini model fallback handling.
- Category-specific design guidance for Portfolio, Restaurant, E-commerce, Blog, Event, Gym, Travel, Startup, Education, and Photography sites.
- Generated HTML, CSS, and JavaScript code viewer with copy actions.
- Sandboxed live preview with desktop/mobile responsive modes.
- Site customization controls for browser title, SEO description, theme color, logo, favicon, and social preview image.
- ZIP export containing `index.html`, Netlify `_headers`, and `_redirects` files.
- Netlify deployment with a unique public URL.
- Clerk sign-in and protected dashboard routes.
- Persistent users, project links, contact submissions, tie-up requests, and payment records.
- Stripe Checkout credit packages with webhook and verification flows.

## Technology Stack

- Next.js 16 App Router
- React 19 with JavaScript and JSX
- Tailwind CSS 4 with PostCSS
- Clerk for authentication and route protection
- Prisma 6 with PostgreSQL
- Google Gemini Generative Language API
- Stripe Checkout and webhooks
- Netlify API for deployments
- Framer Motion and Lucide React for UI

## Requirements

- Node.js 20 or newer is recommended
- npm
- PostgreSQL database
- Clerk application
- Google Gemini API key
- Stripe account for payments
- Netlify personal access token for deployments

## Installation

```bash
npm install
```

Create `.env.local` in the project root. Never commit real secrets.

```env
# Database
DATABASE_URL="postgresql://USER:PASSWORD@HOST:5432/DATABASE?schema=public"

# Clerk
NEXT_PUBLIC_CLERK_PUBLISHABLE_KEY="pk_test_..."
CLERK_SECRET_KEY="sk_test_..."

# Google Gemini
GEMINI_API_KEY="..."
# Optional fallback used by the chat route
GEMINI_API_KEY2="..."

# Supabase compatibility configuration
NEXT_PUBLIC_SUPABASE_URL="https://PROJECT.supabase.co"
NEXT_PUBLIC_SUPABASE_ANON_KEY="..."

# Stripe
STRIPE_SECRET_KEY="sk_test_..."
STRIPE_WEBHOOK_SECRET="whsec_..."

# Application and deployment
NEXT_PUBLIC_APP_URL="http://localhost:3000"
NETLIFY_AUTH_TOKEN="..."
```

Some integrations intentionally degrade when configuration is missing: Prisma and Stripe log warnings and disable related features, while the Supabase client uses no-op auth behavior. Clerk-protected pages still require valid Clerk configuration.

## Database Setup

Generate the Prisma client:

```bash
npx prisma generate
```

Apply the current schema to a development database:

```bash
npx prisma db push
```

For migration-based development:

```bash
npx prisma migrate dev --name init
```

The client is generated into `prisma/generated/client` and loaded through [`lib/prisma.js`](lib/prisma.js).

## Running the App

```bash
npm run dev
```

Open [http://localhost:3000](http://localhost:3000).

| Command | Purpose |
| --- | --- |
| `npm run dev` | Start the development server |
| `npm run build` | Create a production build |
| `npm start` | Start the production server |
| `npm run lint` | Run ESLint |

## Application Routes

### Pages

| Route | Purpose |
| --- | --- |
| `/` | Landing page |
| `/about` | Product information |
| `/contact` | Contact form |
| `/pricing` | Credit package selection |
| `/dashboard` | Authenticated dashboard, credits, and saved projects |
| `/dashboard/new` | Website generation workspace |
| `/projects` | Creator notebook and deployment history |
| `/tieups` | Partnership request form |
| `/x` | Advanced generation workspace with prompt helpers and component blocks |

The root layout configures `ClerkProvider`, shared header/footer components, and `UserSync`.

```mermaid
flowchart TB
    Root["/ layout.js\nClerkProvider, Header, Footer"]
    Root --> Home["/ landing page"]
    Root --> About["/about"]
    Root --> Contact["/contact"]
    Root --> Pricing["/pricing"]
    Root --> Tieups["/tieups"]
    Root --> Gate{"Clerk session\nvalid?"}
    Gate -- "Yes" --> Dash["/dashboard\ncredits, saved projects"]
    Gate -- "No" --> Redirect["Redirect to sign-in"]
    Dash --> New["/dashboard/new\ngeneration workspace"]
    Dash --> Projects["/projects\ndeployment history"]
    Dash --> X["/x\nadvanced workspace"]
```

### API Endpoints

| Method | Endpoint | Purpose |
| --- | --- | --- |
| `POST` | `/api/generate` | Generate or refine HTML/CSS/JS with Gemini |
| `POST` | `/api/chat` | Get prompt-writing assistance |
| `POST` | `/api/auth/user` | Upsert a user profile |
| `GET` | `/api/auth/user?userId=...` | Read current credits |
| `GET` | `/api/projects?userId=...` | List saved project links |
| `POST` | `/api/projects` | Save a project link |
| `DELETE` | `/api/projects?id=...` | Delete a project link |
| `POST` | `/api/contact` | Create a contact submission |
| `GET` | `/api/contact` | Read the latest 50 contact submissions |
| `POST` | `/api/tieups` | Create a partnership request |
| `GET` | `/api/tieups` | Read the latest 50 partnership requests |
| `POST` | `/api/deploy` | Package and deploy generated code to Netlify |
| `POST` | `/api/stripe/checkout` | Create a Stripe Checkout session |
| `GET` | `/api/stripe/verify?session_id=...` | Verify a successful payment |
| `POST` | `/api/stripe/webhook` | Process `checkout.session.completed` |

## System Architecture

The browser client never talks to Gemini, Clerk, Postgres, Stripe, or Netlify directly. Every request goes through the Next.js API routes, which act as the single integration layer and fan out to each backend service.

```mermaid
flowchart TB
    Client["Browser client"]
    API["Next.js API routes\ngenerate, deploy, projects, stripe"]
    Gemini["Gemini API\nsite generation"]
    Clerk["Clerk auth\nsign-in, sessions"]
    DB["Prisma / Postgres\nusers, projects, payments"]
    Pay["Stripe / Netlify\ncredits, deploy"]

    Client --> API
    API --> Gemini
    API --> Clerk
    API --> DB
    API --> Pay
```

- **Client** — the dashboard, `/dashboard/new`, and `/x` workspaces send prompts, poll credits, and render the sandboxed preview iframe.
- **API routes** — a thin server layer that adds category design guidance, calls Gemini, normalizes output, and coordinates persistence, auth, and payments.
- **Gemini** — the generation engine, called with an ordered fallback list of models.
- **Clerk** — protects `/dashboard` and nested routes via `proxy.js` middleware and syncs the signed-in profile through `UserSync`.
- **Prisma / Postgres** — stores `User`, `Contact`, `TieUp`, `ProjectLink`, and `Payment` records.
- **Stripe / Netlify** — Stripe issues and verifies credit purchases; Netlify receives the packaged ZIP and returns a live deployment URL.

## Generation Flow

1. The user selects a category and enters content, style, and functionality requirements.
2. The client sends the combined prompt to `POST /api/generate`.
3. The server adds category design DNA and calls Gemini with the first model in the fallback list.
4. If that model returns a quota or rate-limit error, the server retries with the next model in the list, repeating until a model succeeds or the list is exhausted.
5. Gemini returns `===HTML===`, `===CSS===`, `===JS===`, and `===END===` sections.
6. The server parses and normalizes the response, removes document wrappers, repairs unbalanced CSS braces, and injects a fallback hash router when JavaScript is too short.
7. The workspace renders the result in an iframe and exposes preview, code, copy, export, and deployment actions.

The decision point in step 4 is what makes generation resilient — a single model being rate-limited does not fail the request, it just moves to the next model down the list.

```mermaid
flowchart TB
    Start["User submits prompt"] --> Call["Call current model\nfirst in fallback list"]
    Call --> Check{"Quota or rate-limit\nerror returned?"}
    Check -- "No" --> Parse["Parse HTML / CSS / JS\nsplit on section markers"]
    Parse --> Repair["Repair and normalize\nfix CSS braces, add JS router"]
    Repair --> Preview["Render live preview"]
    Check -- "Yes" --> Next["Try next model\nnext in fallback list"]
    Next -.retry.-> Call
```

- **No** branch: the model responded successfully, so the server moves straight into parsing, repair, and preview rendering.
- **Yes** branch: the current model is out of quota or rate-limited, so the server advances to the next model in `GEMINI_MODELS` and retries the same call. If every model in the list is exhausted, the request fails and an error is returned to the client.

### Site Customization

Before deployment, the builder accepts optional branding details:

- Browser title and brand name
- SEO description
- Theme color exposed as `--criar-brand-color` and `<meta name="theme-color">`
- Brand logo, displayed in the generated site and stored as `brand-logo.*`
- Favicon, with the brand logo used as a fallback when no favicon is supplied
- Social preview image stored as `social-preview.*` and referenced by `og:image`

Images are read in the browser as base64 data URLs, limited to 2 MB each, validated on the server, and written into the deployed Netlify ZIP. Supported formats are PNG, JPEG, WebP, GIF, and ICO. The same branding metadata is included in the local HTML download and live iframe preview.

Generation and refinement credit costs are currently `0` in `app/api/generate/route.js`. Credit and Stripe infrastructure exists for future metering.

```mermaid
flowchart TB
    A["User uploads logo,\nfavicon, social image"] --> B{"File size\nunder 2 MB?"}
    B -- "No" --> C["Reject with size error"]
    B -- "Yes" --> D["Read as base64\ndata URL in browser"]
    D --> E["Validate format on server\nPNG, JPEG, WebP, GIF, ICO"]
    E --> F["Apply branding metadata\ntitle, SEO, theme color"]
    F --> G["Embed in preview,\nHTML download, deploy ZIP"]
```

## Deployment Flow

`POST /api/deploy` removes generator markers, builds `index.html`, adds Netlify `_headers` and `_redirects`, creates a Netlify site, uploads the ZIP archive, and returns the deployed URL. The URL can then be stored with `POST /api/projects` and shown in the dashboard or projects notebook.

```mermaid
flowchart TB
    A["Workspace: user clicks Deploy"] --> B["POST /api/deploy"]
    B --> C["Strip generator markers\nfrom HTML/CSS/JS"]
    C --> D["Build index.html\nwith branding assets"]
    D --> E["Add Netlify _headers\nand _redirects"]
    E --> F["Create Netlify site"]
    F --> G["Upload ZIP archive"]
    G --> H["Return deployed URL"]
    H --> I["POST /api/projects\nsave title + url"]
    I --> J["Shown in dashboard\nand /projects notebook"]
```

## Authentication

Authentication is currently wired through Clerk:

- `ClerkProvider` is configured in `app/layout.js`.
- `proxy.js` protects `/dashboard` and nested routes.
- Client pages use `useUser()` for the signed-in user.
- `app/components/UserSync.jsx` syncs the Clerk profile to the Prisma `User` record.

The Supabase client in [`lib/supabase.js`](lib/supabase.js) is retained for compatibility with earlier auth code. It is not the active route-protection mechanism.

```mermaid
sequenceDiagram
    participant U as Browser
    participant C as Clerk
    participant M as proxy.js middleware
    participant D as Dashboard page
    participant S as UserSync
    participant P as Prisma User

    U->>C: Sign in
    C-->>U: Session token
    U->>M: Request /dashboard
    M->>C: Verify session
    alt Session valid
        C-->>M: Authorized
        M-->>D: Allow request
        D->>S: Mount with Clerk profile
        S->>P: Upsert User record
        P-->>S: Synced profile + credits
        S-->>U: Render dashboard
    else Session invalid
        C-->>M: Unauthorized
        M-->>U: Redirect to sign-in
    end
```

## Prisma Data Model

The complete schema is in [`prisma/schema.prisma`](prisma/schema.prisma).

### `User`

Application profile synchronized from Clerk. Fields include `id`, unique `email`, optional `name` and `avatarUrl`, `credits` defaulting to `100`, and timestamps. Relations: contacts, tie-ups, project links, and payments.

### `Contact`

Contact form submissions with `email`, optional `name` and `subject`, required `message`, timestamp, and optional `User` relation.

### `TieUp`

Partnership requests with company/contact details, proposal text, optional category and website, `status` defaulting to `pending`, timestamps, and optional `User` relation.

### `ProjectLink`

Saved deployment references with `title`, `url`, timestamps, and a required `User` relation.

### `Payment`

Stripe records with unique `stripeSessionId`, optional customer ID, `amount`, `currency`, `status`, optional plan, timestamp, and optional `User` relation.

### Entity Relationship Diagram

```mermaid
erDiagram
    USER ||--o{ CONTACT : submits
    USER ||--o{ TIEUP : requests
    USER ||--o{ PROJECTLINK : saves
    USER ||--o{ PAYMENT : makes

    USER {
        string id PK
        string email
        string name
        string avatarUrl
        int credits
    }
    CONTACT {
        string id PK
        string userId FK
        string email
        string subject
        string message
    }
    TIEUP {
        string id PK
        string userId FK
        string company
        string proposal
        string status
    }
    PROJECTLINK {
        string id PK
        string userId FK
        string title
        string url
    }
    PAYMENT {
        string id PK
        string userId FK
        string stripeSessionId
        int amount
        string currency
        string status
        string plan
    }
```

## Stripe Plans

Checkout uses INR prices and adds credits through the webhook or verification endpoint:

| Plan ID | Display name | Price | Credits |
| --- | --- | ---: | ---: |
| `starter` | Starter Pack | Rs 100 | 100 |
| `pro` | Value Pack | Rs 300 | 400 |
| `enterprise` | Premium Pack | Rs 500 | 700 |

Amounts are sent to Stripe in paise. Configure the webhook endpoint as:

```text
https://YOUR_DOMAIN/api/stripe/webhook
```

For local development, use the Stripe CLI to forward `checkout.session.completed` events to this endpoint and copy the generated signing secret into `STRIPE_WEBHOOK_SECRET`.

```mermaid
sequenceDiagram
    participant U as User
    participant P as /pricing page
    participant A as POST /api/stripe/checkout
    participant S as Stripe Checkout
    participant W as POST /api/stripe/webhook
    participant V as GET /api/stripe/verify
    participant D as Prisma Payment/User

    U->>P: Select a credit package
    P->>A: Create checkout session
    A->>S: Create session for plan
    S-->>A: Session URL
    A-->>U: Redirect to Stripe

    U->>S: Complete payment
    S-->>W: checkout.session.completed event
    W->>D: Record Payment, add credits

    U->>V: Return to app with session_id
    V->>D: Verify session, sync credits
    D-->>U: Updated credit balance
```

## Gemini Model Fallback Strategy

```javascript
const GEMINI_MODELS = [
    'gemini-3.6-flash',
    'gemini-3.5-flash',
    'gemini-3.5-flash-lite',
    'gemini-3.1-flash-lite'
];
```

The list is ordered from highest quality / highest cost down to cheapest / most available. Generation always attempts the best model first and only degrades when forced to by rate limits or quota errors, so most requests never touch anything below the first entry.

### Why each model is in the list, in order

**1. `gemini-3.6-flash` — first choice, best quality-per-token.**
This is Google's current general-purpose Flash workhorse. Google reports it consumes roughly 17% fewer output tokens than 3.5 Flash on the Artificial Analysis Index, with reductions as high as 65% on individual evals like DeepSWE, largely because it reaches a correct answer in fewer reasoning steps and tool calls. For website generation, that means more coherent HTML/CSS/JS output with less wasted generation, which matters directly when the server has to parse `===HTML===/===CSS===/===JS===` sections out of the response.

**2. `gemini-3.5-flash` — second choice, the stable proven tier.**
Google recommends 3.5 Flash as its most intelligent and capable Flash model for general-purpose use. It sits one step below 3.6 in raw capability but is a mature, GA model in the same pricing tier, so falling back to it from 3.6 is a small quality trade-off rather than a steep drop.

**3. `gemini-3.5-flash-lite` — third choice, cheap and fast without being weak.**
This is no longer a "budget" model in the old sense. On several agentic and coding evaluations it now beats the older, larger Gemini 3 Flash outright (SWE-Bench Pro 54.2% vs 49.6%, OSWorld-Verified 74.0%), and it is the fastest model in the 3.5 series at roughly 350 output tokens per second, priced at $0.30 per million input tokens and $2.50 per million output tokens. That combination of speed and decent quality makes it a good third-tier fallback: fast enough that a retry doesn't stall the request, capable enough that output quality doesn't collapse.

**4. `gemini-3.1-flash-lite` — last resort, cheapest and most available.**
The oldest and least capable model in the chain, which is exactly the right role for a final fallback. It is the most cost-efficient model in the Gemini 3 series, built for high-volume workloads, priced around $0.25 per million input tokens and $1.50 per million output tokens. When every better model is exhausted, this tier exists to guarantee the request still returns something instead of failing outright.

### Why this pattern works

- **Degrades quality, not availability.** A user rarely notices whether their site was generated by 3.6 or 3.1 Flash-Lite — they do notice a failed request. The chain trades a small amount of quality for uptime only when necessary.
- **Keeps cost low by default.** Because the first model succeeds most of the time, premium-tier cost is only paid on the rare requests that fall further down the list.
- **Mirrors Google's own recommended tiering.** Google itself suggests treating 3.6 Flash as the primary agent and 3.5 Flash-Lite as a fast execution layer underneath it, which is effectively what this fallback list already does.

> Note: the API endpoints table above still documents an older fallback set (`gemini-3-flash-preview`, `gemini-2.5-flash`, `gemini-2.0-flash`, `gemini-1.5-flash`, `gemini-1.5-flash-8b`) in some earlier project notes. The `GEMINI_MODELS` list shown here is the current, active fallback chain and supersedes those references — the 2.5 and earlier generations trail the 3.x family on reasoning, coding, and agentic benchmarks.

## Project Structure

```text
app/
	api/                 Server routes for generation, auth, data, deploy, and Stripe
	components/          Shared Header, Footer, and UserSync components
	dashboard/           Authenticated dashboard and new-site workflow
	projects/            Saved deployment history
	pricing/             Credit package checkout UI
	tieups/              Partnership request UI
	page.js              Landing page
	layout.js            Root layout and providers
lib/
	prisma.js            Prisma singleton and missing-config fallback
	stripe.js             Lazy Stripe client
	supabase.js           Lazy Supabase compatibility client
prisma/
	schema.prisma        PostgreSQL data model
	generated/client/    Generated Prisma client
public/                Static assets and media
proxy.js               Clerk middleware and protected-route matcher
next.config.mjs        Next.js configuration with React Compiler enabled
```

## Troubleshooting

- `GEMINI_API_KEY not set`: add the key to `.env.local` and restart Next.js.
- `All Gemini models are rate-limited`: check Google AI Studio quota or wait before retrying.
- `Stripe is not configured`: add `STRIPE_SECRET_KEY`; webhook verification also requires `STRIPE_WEBHOOK_SECRET`.
- Database operations return empty data: verify `DATABASE_URL`, run `npx prisma generate`, and apply the schema.
- Deployment reports a missing Netlify token: add `NETLIFY_AUTH_TOKEN`.
- Clerk redirects unexpectedly: verify both Clerk keys and the active Clerk instance configuration.

## Notes and Known Caveats

- The database schema uses PostgreSQL and the generated Prisma client is checked under `prisma/generated/client`.
- The generator currently assigns free credit costs (`0`) even though the UI and pricing pages expose credit packages.
- The dashboard and middleware use Clerk, while older project notes refer to Supabase Auth. Clerk is the current authentication source of truth.
- API routes accept user IDs from request bodies and query parameters. Before production use, add server-side authorization checks so users cannot read or modify another user's records.
- Contact and tie-up `GET` endpoints currently return recent records without an admin authorization check; protect these endpoints before exposing them publicly.

## Useful Documentation

- [Next.js App Router](https://nextjs.org/docs/app)
- [Clerk Next.js documentation](https://clerk.com/docs/quickstarts/nextjs)
- [Prisma documentation](https://www.prisma.io/docs)
- [Google Gemini API documentation](https://ai.google.dev/gemini-api/docs)
- [Stripe Checkout documentation](https://docs.stripe.com/checkout)
- [Netlify API documentation](https://docs.netlify.com/api/get-started/)

First, run the development server:

```bash
npm run dev
# or
yarn dev
# or
pnpm dev
# or
bun dev
```

Open [http://localhost:3000](http://localhost:3000) with your browser to see the result.

You can start editing the page by modifying `app/page.js`. The page auto-updates as you edit the file.

This project uses [`next/font`](https://nextjs.org/docs/app/building-your-application/optimizing/fonts) to automatically optimize and load [Geist](https://vercel.com/font), a new font family for Vercel.

## Learn More

To learn more about Next.js, take a look at the following resources:

- [Next.js Documentation](https://nextjs.org/docs) - learn about Next.js features and API.
- [Learn Next.js](https://nextjs.org/learn) - an interactive Next.js tutorial.

You can check out [the Next.js GitHub repository](https://github.com/vercel/next.js) - your feedback and contributions are welcome!

## Deploy on Vercel

The easiest way to deploy your Next.js app is to use the [Vercel Platform](https://vercel.com/new?utm_medium=default-template&filter=next.js&utm_source=create-next-app&utm_campaign=create-next-app-readme) from the creators of Next.js.

Check out our [Next.js deployment documentation](https://nextjs.org/docs/app/building-your-application/deploying) for more details.
