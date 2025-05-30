## 1. Executive Summary

Redesign the PACE website into a mobile-first, responsive, and highly performant frontend that:

* **Maximizes fundraising** via Stripe-powered donations and sponsorships
* **Surfaces programs** with rich storytelling and impact metrics
* **Enables partner & volunteer engagement** through streamlined sign-up flows
* **Empowers admins** with a custom, lightweight CMS in the PACE Admin Dashboard
* **Leverages Supabase** for real-time data, role-based access, and seamless auth

---

## 2. Goals & Success Metrics

| Goal                                | Success Metric                                       |
| ----------------------------------- | ---------------------------------------------------- |
| Increase total donations by 25%     | # of Stripe transactions; \$ volume YOY              |
| Boost program sponsorships          | # of corporate sponsorships; avg. sponsorship amount |
| Grow volunteer sign-ups by 40%      | Volunteer form submissions                           |
| Improve content update velocity     | Admin CMS: time to publish new page ≤ 5 minutes      |
| Achieve top-tier performance scores | Lighthouse ≥ 90 (mobile & desktop)                   |

---

## 3. Target Users & Personas

| Persona                 | Description                                                    | Key Journeys                                  |
| ----------------------- | -------------------------------------------------------------- | --------------------------------------------- |
| **Donor**               | Individual seeking to make one-time or recurring contributions | Find “Donate,” complete quick form            |
| **Sponsor (Corporate)** | CSR lead evaluating program sponsorship opportunities          | Review Programs → Sponsor → download Pkg      |
| **Partner**             | NGO/academic institution interested in collaboration           | Explore “Partner With Us,” submit RFP         |
| **Volunteer**           | Skilled professional offering time/skills                      | Browse “Volunteer,” view opportunities        |
| **Admin**               | PACE staff managing content, programs, and user inquiries      | Login to Admin Dash → Edit CMS → View metrics |

---

## 4. Information Architecture

1. **Home**

   * Hero with mission pillars
   * Featured Fundraisers carousel
   * Live Impact Counters
   * Call-out: BLACK KIDS DECODED
   * Blog highlights
   * “Donate” CTA
2. **About**

   * Mission & history
   * Leadership bios
   * 501(c)(3) & EIN
3. **Programs**

   * Program tiles (Union, AfroDetox, Koya Academy…)
   * Dedicated detail pages
4. **Media**

   * Blog index (CMS-driven)
   * Press & gallery
5. **Get Involved**

   * Partner inquiry form
   * Volunteer opportunities listing
6. **Donate**

   * One-time vs. recurring (Stripe integration)
   * Sponsorship tiers & benefits
7. **Contact**

   * Contact form (Supabase + email trigger)
   * Newsletter sign-up

---

## 5. Technical Architecture

### 5.1 Frontend

* **Framework**: Next.js (React)
* **Styling**: Tailwind CSS
* **Animation**: Framer Motion for micro-interactions
* **Hosting**: Vercel (Global CDN)

### 5.2 Backend Services

* **Database & Auth**: Supabase

  * **Auth**: Email/password + OAuth for Admins
  * **Row-level Security**: Roles (`public`, `donor`, `sponsor`, `partner`, `volunteer`, `admin`)
* **Payments**: Stripe

  * **Checkout**: Stripe Elements for seamless embeds
  * **Webhooks**: Supabase Edge Function to record successful payments
* **CMS**: Custom tables in Supabase + Admin UI

  * **Content Types**: `pages`, `blog_posts`, `programs`, `faqs`, `impact_metrics`
  * **Media**: Supabase Storage for images/videos
* **Email**: SendGrid (via Supabase Functions) for notifications

---

## 6. Data Model & Schema (Supabase)

```sql
-- Users & Roles
users (
  id           uuid PK,
  email        text UNIQUE,
  full_name    text,
  role         enum('public','donor','sponsor','partner','volunteer','admin'),
  created_at   timestamp
);

-- Programs
programs (
  id           uuid PK,
  title        text,
  slug         text UNIQUE,
  description  text,
  hero_image   text (storage path),
  created_by   uuid → users.id,
  created_at   timestamp
);

-- Donations & Sponsorships
donations (
  id              uuid PK,
  user_id         uuid → users.id,
  stripe_charge   text,
  amount_cents    int,
  type            enum('one_time','recurring','sponsorship'),
  program_id      uuid → programs.id NULL,
  created_at      timestamp
);

-- Volunteer Applications
volunteer_apps (
  id           uuid PK,
  user_id      uuid → users.id,
  program_id   uuid → programs.id NULL,
  message      text,
  status       enum('pending','approved','rejected'),
  created_at   timestamp
);

-- Partnerships / Inquiries
partnerships (
  id           uuid PK,
  user_id      uuid → users.id,
  organization text,
  message      text,
  status       enum('pending','engaged','closed'),
  created_at   timestamp
);

-- CMS Content
pages (
  id           uuid PK,
  slug         text UNIQUE,
  title        text,
  body_md      text,
  metadata     jsonb,
  updated_by   uuid → users.id,
  updated_at   timestamp
);

blog_posts (
  id           uuid PK,
  title        text,
  slug         text UNIQUE,
  excerpt      text,
  body_md      text,
  author_id    uuid → users.id,
  published_at timestamp,
  status       enum('draft','published'),
  created_at   timestamp,
  updated_at   timestamp
);

impact_metrics (
  id           uuid PK,
  label        text,
  value        int,
  icon         text (storage path),
  updated_at   timestamp
);
```

---

## 7. Functional Requirements

| Feature                       | Details                                                                                 |
| ----------------------------- | --------------------------------------------------------------------------------------- |
| **Authentication**            | Supabase Auth for Admins; public flows for donors/volunteers                            |
| **Donations Checkout**        | Stripe Elements; choose one-time/recurring; assign to program (optional)                |
| **Admin Dashboard**           | CRUD for `pages`, `programs`, `blog_posts`, `impact_metrics`; media uploader            |
| **Dynamic Content Loading**   | ISR (Incremental Static Regeneration) for public pages; real-time counters via Supabase |
| **Forms & Validation**        | React Hook Form + Zod; CAPTCHA on public forms; inline error handling                   |
| **Role-based Access Control** | RLS policies to prevent unauthorized data access                                        |
| **Performance**               | Image lazy-loading; prefetching; code splitting; Lighthouse ≥ 90                        |
| **Accessibility**             | WCAG 2.1 AA compliance; semantic HTML; keyboard nav; ARIA labels                        |

---

## 8. Design System & Visual Style

* **Color Palette**

  * Navy Blue: #1A1F71
  * Teal Accent: #00C2A0
  * Gold Highlight: #F5A623
  * Neutral Gray: #F7F7F7 / #333333
* **Typography**

  * Headings: Montserrat (700 / 400)
  * Body: Inter (400 / 500)
* **Spacing & Layout**

  * 4-column grid on desktop; single-col stack on mobile
  * Consistent 8px base spacing scale
* **Components**

  * Buttons (primary, secondary, outlined)
  * Cards (programs, blog teasers)
  * Modals (donate, form confirmations)
  * Carousel (fundraisers)
* **Motion**

  * Fade/slide on scroll into view
  * Micro-interactions on hover/focus

---

## 9. Analytics & Tracking

* **Google Analytics 4**: pageviews, Donate clicks, form submits
* **Stripe Dashboard**: transaction volume, recurring retention
* **Hotjar**: heatmaps on Donate & Get Involved pages
* **Supabase Metrics**: function invocations, RLS errors

---

## 10. Timeline & Milestones

| Phase                       | Duration | Deliverables                                          |
| --------------------------- | -------- | ----------------------------------------------------- |
| **Discovery & IA**          | 1 week   | Final IA diagram, detailed wireframes                 |
| **Design & Prototyping**    | 2 weeks  | Hi-fi mockups, design system library                  |
| **Frontend Dev & Supabase** | 4 weeks  | Next.js pages, Supabase schema & RLS policies, CMS UI |
| **Stripe Integration**      | 1 week   | Checkout flows, webhooks, edge functions              |
| **QA & Accessibility**      | 1 week   | Cross-browser testing, WCAG audit report              |
| **Launch & Monitoring**     | 1 week   | DNS cutover, performance tuning, analytics setup      |

---
