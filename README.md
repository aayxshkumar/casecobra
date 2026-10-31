# casecobra

![CI](https://github.com/aayxshkumar/casecobra/actions/workflows/ci.yml/badge.svg)

> Learning project from June 2024, built while learning Next.js, Prisma and Kinde auth.

A custom phone case shop: upload an image, place it on a phone case, pick options and pay with Stripe.

## What it does

- Upload (`/configure/upload`): drag and drop a PNG or JPG, stored with UploadThing.
- Design (`/configure/design`): move and resize the image on a phone template, then choose a colour (black, blue, rose), an iPhone model (X to 15), a material (silicone or polycarbonate) and a finish (smooth or textured). The cropped image is uploaded and the choices are saved with Prisma.
- Preview (`/configure/preview`): shows the case and the price (base price of $14, plus $5 for polycarbonate and $3 for a textured finish). Checking out asks signed-out visitors to log in with Kinde first, then creates a Stripe Checkout session.
- Stripe webhook (`/api/webhooks`): on `checkout.session.completed`, marks the order as paid, stores the shipping and billing addresses and sends an order confirmation email through Resend (built with React Email).
- Thank you page (`/thank-you`): polls the order until the payment is confirmed.
- Admin dashboard (`/dashboard`): only for the account whose email matches `ADMIN_EMAIL`. Shows revenue for the last week and month against fixed goals, lists orders and lets the admin change an order's shipping status.

## Tech stack

- Next.js 15 (App Router, server actions), React 18, TypeScript
- Tailwind CSS, shadcn/ui components (Radix UI), Framer Motion, lucide-react
- PostgreSQL with Prisma
- Kinde for authentication
- Stripe Checkout and webhooks
- UploadThing and sharp for image uploads, react-rnd for positioning
- TanStack Query, Zod, Resend and React Email

## Getting started

Requires Node.js 22 (the version used in CI) and a PostgreSQL database.

```bash
git clone https://github.com/aayxshkumar/casecobra.git
cd casecobra
npm ci                       # also runs prisma generate (postinstall)
cp .env.example .env         # then fill in your values
npm run dev
```

Open http://localhost:3000.

There are no Prisma migrations in the repository, so the tables have to be created from `prisma/schema.prisma` against your own database (Prisma's `db push` does this).

Other scripts:

```bash
npx prisma generate
npm run lint
npm run build
npm start
```

### Environment variables

| Name | Used for |
| --- | --- |
| `DATABASE_URL` | PostgreSQL connection string for Prisma |
| `KINDE_CLIENT_ID`, `KINDE_CLIENT_SECRET`, `KINDE_ISSUER_URL` | Kinde application credentials |
| `KINDE_SITE_URL`, `KINDE_POST_LOGOUT_REDIRECT_URL`, `KINDE_POST_LOGIN_REDIRECT_URL` | Kinde redirect URLs; `/auth-callback` is the page meant for the post-login redirect |
| `ADMIN_EMAIL` | Email of the account that can open `/dashboard` |
| `NEXT_PUBLIC_SERVER_URL` | Base URL for the Stripe Checkout success and cancel URLs |
| `STRIPE_SECRET_KEY` | Stripe API calls |
| `STRIPE_WEBHOOK_SECRET` | Verifying Stripe webhook signatures |
| `RESEND_API_KEY` | Sending the order confirmation email |
| `UPLOADTHING_SECRET` | UploadThing image uploads |

`npm run build` does not need working services, but the Kinde SDK and the Resend client need values present at build time. CI builds with placeholder values; see `.github/workflows/ci.yml`.
