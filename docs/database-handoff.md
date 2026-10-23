# EKSU Jobber database handoff

The current frontend uses demonstration data. Replace that data through a service layer when your database is ready; do not put database credentials or admin keys in the browser.

## Required entities

| Entity | Required fields | Notes |
| --- | --- | --- |
| `profiles` | `id`, `full_name`, `avatar_url`, `role`, `location`, `created_at` | `id` should reference the authentication user. Roles: `member`, `admin`. |
| `categories` | `id`, `name`, `slug`, `is_active`, `position` | Keep slugs stable for URLs and filters. |
| `listings` | `id`, `seller_id`, `category_id`, `title`, `description`, `price`, `price_type`, `condition`, `location`, `status`, `created_at` | Status: `draft`, `pending`, `active`, `rejected`, `archived`, `sold`. |
| `listing_images` | `id`, `listing_id`, `storage_path`, `alt_text`, `position` | Store listing images in private or controlled storage. |
| `saved_listings` | `user_id`, `listing_id`, `created_at` | Unique pair: `user_id`, `listing_id`. |
| `conversations` | `id`, `listing_id`, `buyer_id`, `seller_id`, `updated_at` | One conversation per buyer, seller and listing. |
| `messages` | `id`, `conversation_id`, `sender_id`, `body`, `created_at`, `read_at` | Do not expose a conversation to anyone outside its participants or admins. |
| `reports` | `id`, `reporter_id`, `listing_id`, `reason`, `details`, `status`, `reviewed_by`, `reviewed_at` | Status: `open`, `in_review`, `resolved`, `dismissed`. |
| `reviews` | `id`, `listing_id`, `reviewer_id`, `seller_id`, `rating`, `comment`, `created_at` | Only permit reviews after a completed transaction if you add transactions. |
| `admin_actions` | `id`, `admin_id`, `target_type`, `target_id`, `action`, `note`, `created_at` | Audit approvals, rejections, suspensions and report decisions. |

## Minimum security rules

- A signed-in user can create a listing only with their own `seller_id`.
- A seller can edit or archive only their own listing.
- Public browsing must return only listings where `status = 'active'`.
- New listings must begin as `draft` or `pending`; clients must never set them to `active` directly.
- Saved items belong only to the signed-in user.
- Only conversation participants can read or create messages in that conversation.
- Only an `admin` can approve, reject, hide or restore a listing, resolve a report, or view the complete admin queue.
- Never trust a role sent by the browser. Check it in database policies or server-side code.
- Keep payment provider keys and service-role keys on the server only.

## First API operations to implement

1. `GET /listings` with search, category, condition, price and pagination.
2. `GET /listings/:id` for an active listing.
3. `POST /listings` to create a pending listing.
4. `PATCH /listings/:id` for the listing owner.
5. `POST /listing-images` for controlled image uploads.
6. `GET` and `POST /saved-listings` for the signed-in user.
7. `GET` and `POST /conversations` and `/messages`.
8. Admin-only `GET /admin/listings?status=pending`, plus approve/reject actions.
9. `POST /reports` and admin-only report review actions.

## Recommended implementation order

1. Authentication and profiles.
2. Categories, listings, image storage and public browse.
3. Seller dashboard and listing moderation.
4. Saved listings and messaging.
5. Reports and administration audit trail.
6. Reviews, transactions and payments.

## Frontend integration boundary

Create a `src/services/marketplace.js` module when you begin the connection. Keep all database calls there, then replace the demonstration arrays in `src/App.jsx` gradually. This prevents database code from being scattered through page components.
