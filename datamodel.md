## 1. `users`

| Column       | Type                       | Constraints                      | Description                           |
| ------------ | -------------------------- | -------------------------------- | ------------------------------------- |
| `id`         | `uuid`                     | PK, default `uuid_generate_v4()` | Unique user identifier                |
| `email`      | `text`                     | UNIQUE, NOT NULL                 | Login email                           |
| `full_name`  | `text`                     | NOT NULL                         | User’s display name                   |
| `role`       | `user_role` (enum)         | NOT NULL, default `'public'`     | `public` / `donor` / `sponsor` / etc. |
| `created_at` | `timestamp with time zone` | NOT NULL, default `now()`        | Account creation timestamp            |

**Enum**:

```sql
CREATE TYPE user_role AS ENUM (
  'public','donor','sponsor','partner','volunteer','admin'
);
```

---

## 2. `programs`

| Column        | Type                       | Constraints                      | Description                |
| ------------- | -------------------------- | -------------------------------- | -------------------------- |
| `id`          | `uuid`                     | PK, default `uuid_generate_v4()` | Program identifier         |
| `title`       | `text`                     | NOT NULL                         | Program name               |
| `slug`        | `text`                     | UNIQUE, NOT NULL                 | URL-friendly key           |
| `description` | `text`                     | NOT NULL                         | Rich text or Markdown body |
| `hero_image`  | `text`                     | —                                | Supabase storage path      |
| `created_by`  | `uuid`                     | FK → `users.id`, NOT NULL        | Admin who created it       |
| `created_at`  | `timestamp with time zone` | NOT NULL, default `now()`        | Creation timestamp         |

---

## 3. `donations`

| Column          | Type                       | Constraints                      | Description                              |
| --------------- | -------------------------- | -------------------------------- | ---------------------------------------- |
| `id`            | `uuid`                     | PK, default `uuid_generate_v4()` | Donation record ID                       |
| `user_id`       | `uuid`                     | FK → `users.id`, NOT NULL        | Who gave                                 |
| `stripe_charge` | `text`                     | NOT NULL                         | Stripe charge or payment intent ID       |
| `amount_cents`  | `integer`                  | NOT NULL                         | Donation amount in cents                 |
| `type`          | `donation_type` (enum)     | NOT NULL                         | `one_time` / `recurring` / `sponsorship` |
| `program_id`    | `uuid`                     | FK → `programs.id`, NULLABLE     | Optional linkage to a program            |
| `created_at`    | `timestamp with time zone` | NOT NULL, default `now()`        | When it occurred                         |

**Enum**:

```sql
CREATE TYPE donation_type AS ENUM (
  'one_time','recurring','sponsorship'
);
```

---

## 4. `volunteer_apps`

| Column       | Type                       | Constraints                      | Description                         |
| ------------ | -------------------------- | -------------------------------- | ----------------------------------- |
| `id`         | `uuid`                     | PK, default `uuid_generate_v4()` | Application ID                      |
| `user_id`    | `uuid`                     | FK → `users.id`, NOT NULL        | Applicant                           |
| `program_id` | `uuid`                     | FK → `programs.id`, NULLABLE     | Program of interest                 |
| `message`    | `text`                     | —                                | Cover note or motivation text       |
| `status`     | `app_status` (enum)        | NOT NULL, default `'pending'`    | `pending` / `approved` / `rejected` |
| `created_at` | `timestamp with time zone` | NOT NULL, default `now()`        | Submission timestamp                |

**Enum**:

```sql
CREATE TYPE app_status AS ENUM (
  'pending','approved','rejected'
);
```

---

## 5. `partnerships`

| Column         | Type                       | Constraints                      | Description                      |
| -------------- | -------------------------- | -------------------------------- | -------------------------------- |
| `id`           | `uuid`                     | PK, default `uuid_generate_v4()` | Inquiry ID                       |
| `user_id`      | `uuid`                     | FK → `users.id`, NOT NULL        | Who inquired                     |
| `organization` | `text`                     | NOT NULL                         | Partner org name                 |
| `message`      | `text`                     | —                                | Inquiry details                  |
| `status`       | `part_status` (enum)       | NOT NULL, default `'pending'`    | `pending` / `engaged` / `closed` |
| `created_at`   | `timestamp with time zone` | NOT NULL, default `now()`        | Submitted at                     |

**Enum**:

```sql
CREATE TYPE part_status AS ENUM (
  'pending','engaged','closed'
);
```

---

## 6. `pages` (Custom CMS)

| Column       | Type                       | Constraints                      | Description                        |
| ------------ | -------------------------- | -------------------------------- | ---------------------------------- |
| `id`         | `uuid`                     | PK, default `uuid_generate_v4()` | Page ID                            |
| `slug`       | `text`                     | UNIQUE, NOT NULL                 | Path segment (e.g., “about”)       |
| `title`      | `text`                     | NOT NULL                         | Page title                         |
| `body_md`    | `text`                     | NOT NULL                         | Markdown content                   |
| `metadata`   | `jsonb`                    | —                                | SEO tags, open-graph, custom flags |
| `updated_by` | `uuid`                     | FK → `users.id`, NOT NULL        | Last editor                        |
| `updated_at` | `timestamp with time zone` | NOT NULL, default `now()`        | Last publish/update                |

---

## 7. `blog_posts`

| Column         | Type                       | Constraints                      | Description           |
| -------------- | -------------------------- | -------------------------------- | --------------------- |
| `id`           | `uuid`                     | PK, default `uuid_generate_v4()` | Post ID               |
| `title`        | `text`                     | NOT NULL                         | Headline              |
| `slug`         | `text`                     | UNIQUE, NOT NULL                 | URL key               |
| `excerpt`      | `text`                     | —                                | Teaser text           |
| `body_md`      | `text`                     | NOT NULL                         | Full content          |
| `author_id`    | `uuid`                     | FK → `users.id`, NOT NULL        | Writer                |
| `published_at` | `timestamp with time zone` | —                                | Goes live when set    |
| `status`       | `post_status` (enum)       | NOT NULL, default `'draft'`      | `draft` / `published` |
| `created_at`   | `timestamp with time zone` | NOT NULL, default `now()`        | Creation timestamp    |
| `updated_at`   | `timestamp with time zone` | NOT NULL, default `now()`        | Last edit timestamp   |

**Enum**:

```sql
CREATE TYPE post_status AS ENUM (
  'draft','published'
);
```

---

## 8. `impact_metrics`

| Column       | Type                       | Constraints                      | Description                 |
| ------------ | -------------------------- | -------------------------------- | --------------------------- |
| `id`         | `uuid`                     | PK, default `uuid_generate_v4()` | Metric record ID            |
| `label`      | `text`                     | NOT NULL                         | e.g. “Hours Served”         |
| `value`      | `integer`                  | NOT NULL                         | Numeric total               |
| `icon`       | `text`                     | —                                | Storage path for icon asset |
| `updated_at` | `timestamp with time zone` | NOT NULL, default `now()`        | Last value update           |

---

## Relationships & Cardinalities

* **`users` 1––< `donations`**
* **`programs` 1––< `donations`**
* **`users` 1––< `volunteer_apps`**, **`programs` 1––< `volunteer_apps`**
* **`users` 1––< `partnerships`**
* **`users` 1––< `pages`**, **`users` 1––< `blog_posts`**
* **No direct FKs between `impact_metrics` and other tables** (read‐only display data)

---

### Sample Supabase RLS Snippet (e.g., `donations`)

```sql
-- Only the owning donor or admins can view a donation
CREATE POLICY "donor_can_view_own"
  ON donations
  FOR SELECT USING (
    auth.role() = 'admin'
    OR user_id = auth.uid()
  );
```

---
