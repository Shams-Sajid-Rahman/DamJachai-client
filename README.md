# DamJachai (দাম যাচাই)

The web app for **DamJachai** ("Check the price"): compare the price and quality signals of diagnostic tests
(blood tests, X-ray, ultrasound, MRI, CT and more) across diagnostic labs in Bangladeshi cities.

The Express API lives in a separate repository: **[DamJachai-server](https://github.com/Shams-Sajid-Rahman/DamJachai-server)**.
This app needs it running to show any data.

> DamJachai does not give medical advice. Prices can change, so always confirm with the lab.

## Features

- **Search** tests in Bangla or English, or by an alias such as "CBC", "HbA1c" or "ডেঙ্গু NS1".
- **Compare** every lab that offers a test. Filter by city, area, home collection, accreditation and minimum rating, and sort by price,
  rating or report time. The cheapest lab is highlighted in coral with a "Save ৳X" badge. Prices older than 90 days are
  marked, and a price history chart (Recharts) shows how each lab's price changed.
- **Basket**: add several tests and see the total at each lab, cheapest first, plus the labs that miss a test.
- **Contribute**: logged-in users report prices and write reviews. Both wait for an admin. Users can also save favorite labs.
- **Admin panel**: dashboard with charts, full create/edit/delete for labs, tests, categories, cities, areas and prices,
  CSV import with preview, moderation queues, user management, and a filterable audit log.
- **3D landing page** (React Three Fiber) with a scroll story, live data from the API, and fallbacks for reduced motion and weak devices.
- **Bangla and English** everywhere, with Bengali digits (৳১,২০০) in Bangla. The language choice is remembered.

## Tech stack

| Part | Tools |
|------|-------|
| Framework | React 19 + Vite 6, React Router 7 |
| Data | Axios, TanStack Query 5 |
| Styling and i18n | Tailwind CSS 3, react-i18next |
| Charts | Recharts 3 |
| 3D and motion | three.js, @react-three/fiber 9, @react-three/drei 10, Framer Motion 12 |
| Tests | Vitest 4 + Testing Library, jsdom 26 |

## Project structure

```
DamJachai-client/
├── public/                favicon, robots.txt
├── src/
│   ├── components/        ui/ (Button, Input, Select, Card, Badge, Modal, Table, Pagination, Toast, ...),
│   │                      layout/ (Navbar, Footer), charts/, SearchBar, PriceTag, RatingStars, ...
│   ├── context/           AuthContext, BasketContext
│   ├── hooks/             TanStack Query hooks, URL state, page meta, ...
│   ├── i18n/ locales/     i18next setup; bn/ and en/ JSON per feature
│   ├── lib/               axios instance + refresh interceptor, API clients, formatting, device tier
│   ├── pages/             landing/ (3D scene), public/, auth/, admin/
│   ├── routes/            router and guards (RequireAuth, RequireAdmin, GuestOnly)
│   └── test/              Vitest tests
└── docs/                  landing-performance.md
```

## Requirements

- Node.js **20+** and npm 10+
- The [DamJachai API](https://github.com/Shams-Sajid-Rahman/DamJachai-server) running locally or deployed

## Setup

```bash
npm install
cp .env.example .env     # VITE_API_URL points at the API
npm run dev              # http://localhost:5173
```

On Windows (Command Prompt), use `copy .env.example .env` instead of `cp`.

Start the API first. To open `/admin`, log in with the admin account created by the API's `npm run seed`.

### Scripts

| Command | What it does |
|---------|--------------|
| `npm run dev` | Starts the Vite dev server on port 5173 |
| `npm run build` | Production build into `dist/` |
| `npm run preview` | Serves the production build locally |
| `npm test` | Runs the UI tests once |
| `npm run test:watch` | Runs the UI tests in watch mode |

### Environment variables (`.env`)

| Name | Example | Notes |
|------|---------|-------|
| `VITE_API_URL` | `http://localhost:5000/api` | Base URL of the API. Set it to the deployed API URL when deploying. It is baked in at build time. |

When the app and API are deployed on different domains, the API needs `CLIENT_URL` set to this app's URL and
`COOKIE_SAMESITE=none` (with HTTPS), or login sessions won't survive a page reload.

## Testing

```bash
npm test
```

**28 Vitest + Testing Library tests** (`src/test`). They cover:
- every route rendering in both languages
- guards for `/favorites` and `/admin`
- landing: headline outside the canvas, 2D fallback without WebGL, live API data
- compare highlighting and filters, basket totals, search, and the formatting helpers

## Security notes

- The access token is kept in memory only (never `localStorage`). The refresh token is an httpOnly cookie set by the API,
  and the axios interceptor refreshes the access token when it expires.
- The admin pages are guarded on the client, but the real protection is on the API (`requireAuth` + `requireRole('admin')`).

## Performance

The landing page loads three.js lazily, in its own chunk, and only when the device can handle it.
Lighthouse on the production build: **mobile 81**, **desktop 95** for performance, and 100 for accessibility, best
practices and SEO. Details in [docs/landing-performance.md](docs/landing-performance.md).

## Known issues and limits

1. **Mobile LCP is about 4.5 s** on Lighthouse's slow-4G phone profile, because the page is rendered in the browser.
   Pre-rendering the landing HTML (SSG/SSR) would fix it.
2. **The lab picker on "Report a price"** loads at most 100 labs. A searchable picker will be needed once there are more labs.
3. **The Node version matters:** the project targets Node 20.13. Vite 7, Vitest 5 and jsdom 27+ need newer Node
   versions, so the versions are pinned (Vite 6, Vitest 4, jsdom 26 through an npm override in `package.json`).
   Some packages (such as react-i18next 18) print an engine warning on Node 20; it still works.
