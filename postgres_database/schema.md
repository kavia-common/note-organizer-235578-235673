# Notes App PostgreSQL Schema (step 01.02)

This container uses `db_connection.txt` as the authoritative connection string.

## Connection

`db_connection.txt` contains a command like:

- `psql postgresql://appuser:dbuser123@localhost:5000/myapp`

When connecting programmatically from the backend, use the URL portion:

- `postgresql://appuser:dbuser123@localhost:5000/myapp`

## Extensions

- `pgcrypto` is enabled (used for `gen_random_uuid()` UUID generation).

## Tables

### `users`
- `id` UUID PK, default `gen_random_uuid()`
- `email` unique, required
- `password_hash` required (currently seeded with a placeholder value for demo user)
- `display_name` optional
- `created_at`, `updated_at` (auto-maintained)

### `notes`
- `id` UUID PK, default `gen_random_uuid()`
- `user_id` FK → `users(id)` ON DELETE CASCADE
- `title` (defaults to empty string)
- `content` (defaults to empty string)
- `is_pinned` boolean
- `is_favorited` boolean
- `created_at`, `updated_at` (auto-maintained)

### `tags`
- `id` UUID PK, default `gen_random_uuid()`
- `user_id` FK → `users(id)` ON DELETE CASCADE
- `name` required
- `created_at`
- Unique constraint: `(user_id, name)` (tags are unique per user)

### `note_tags`
Join table for many-to-many between notes and tags.
- `note_id` FK → `notes(id)` ON DELETE CASCADE
- `tag_id` FK → `tags(id)` ON DELETE CASCADE
- `created_at`
- Primary key: `(note_id, tag_id)`

## Indexes
- `idx_notes_user_updated_at` on `(user_id, updated_at DESC)`
- `idx_notes_user_pinned_updated_at` on `(user_id, is_pinned DESC, updated_at DESC)`
- `idx_note_tags_tag_id` on `(tag_id)`

## Triggers
A single trigger function:
- `set_updated_at()` sets `NEW.updated_at = now()` on updates

Triggers:
- `trg_users_updated_at` on `users`
- `trg_notes_updated_at` on `notes`

## Seed data
Minimal seed data inserted:
- User: `demo@example.com` / display name `Demo User`
- Note: `Welcome` (pinned + favorited)
- Tag: `inbox`
- Relation: Welcome note tagged with inbox

All seed INSERTs are idempotent via `ON CONFLICT DO NOTHING` (or unique constraints).
