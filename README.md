# Simon Mnjoroge Bookshop
*"A Haven of Knowledge"*

A full PHP + MySQL online bookshop: user accounts, book catalog with
admin uploads, cart, checkout, and payment via **Stripe (card)** and
**M-Pesa (STK Push)**, plus Cash on Delivery.

Mission: *To be the leading wholistic partner for individuals able to
face life's challenges.*

---

## 1. Requirements

- PHP 8.0+ with extensions: `pdo_mysql`, `curl`, `fileinfo`
- MySQL 5.7+ / MariaDB 10.3+
- Apache with `mod_rewrite`/`mod_headers` (or Nginx — see note below)
- An SSL certificate for production (required by Stripe & M-Pesa)

## 2. Installation

1. **Upload the files** to your web host (e.g. `public_html/`) or place them
   in your local server's document root (XAMPP/WAMP/MAMP `htdocs`).

2. **Create the database.** Import the schema:
   ```bash
   mysql -u root -p < sql/schema.sql
   ```
   This creates the `smb_bookshop` database and all tables, plus starter
   categories. No admin account is created yet (see step 4).

3. **Configure the site.** Open `config/config.php` and set:
   - `DB_HOST`, `DB_NAME`, `DB_USER`, `DB_PASS` — your MySQL credentials
   - `SITE_URL` — your domain, no trailing slash (e.g. `https://simonmnjorogebookshop.com`)
   - `STRIPE_SECRET_KEY`, `STRIPE_PUBLISHABLE_KEY`, `STRIPE_WEBHOOK_SECRET` — from https://dashboard.stripe.com/apikeys
   - `MPESA_*` constants — from https://developer.safaricom.co.ke (Daraja API)

4. **Create your admin account.** Visit:
   ```
   https://yourdomain.com/setup/create_admin.php
   ```
   Fill in your name, email, and password. **Then delete `setup/create_admin.php`**
   (or move it outside the web root) — it refuses to run again once an
   admin exists, but deleting it is best practice.

5. **Make uploads writable:**
   ```bash
   chmod -R 755 uploads/
   ```

6. Log in at `/login.php` with your admin account, then go to **Admin →
   Add New Book** to start listing books.

## 3. Payment setup

### Stripe (card payments)
1. Get your API keys from the Stripe Dashboard and add them to `config/config.php`.
2. In Stripe Dashboard → Developers → Webhooks, add an endpoint:
   `https://yourdomain.com/payment/stripe_webhook.php`, subscribed to
   `checkout.session.completed`. Copy the signing secret into
   `STRIPE_WEBHOOK_SECRET`.
3. The webhook is what actually marks an order "paid" — this is safer
   than trusting the browser redirect alone.
4. Test with Stripe's test card `4242 4242 4242 4242`, any future
   expiry, any CVC, while using your `sk_test_...` key.

### M-Pesa (STK Push)
1. Register an app at the Safaricom Daraja portal to get your consumer
   key/secret, shortcode, and passkey.
2. Set `MPESA_CALLBACK_URL` to a **publicly reachable HTTPS URL**
   (Safaricom cannot call `localhost`) pointing at
   `payment/mpesa_callback.php`. For local testing, use a tunnel tool
   like ngrok.
3. Start in `MPESA_ENV = 'sandbox'`, switch to `'production'` and your
   live shortcode/passkey when Safaricom approves you.

### Cash on Delivery
Works out of the box — no configuration needed. Orders are marked
`pending` until an admin updates the status after physical payment.

## 4. Project structure

```
config/         Site + DB configuration
includes/       Shared header/footer/navbar/helper functions
payment/        Stripe + M-Pesa integration
admin/          Admin dashboard, book & order management
uploads/        Book covers and eBook files (created on upload)
sql/schema.sql  Database schema
setup/          One-time admin creation script (delete after use)
```

## 5. Security notes

- Passwords are hashed with PHP's `password_hash()` (bcrypt).
- All forms are protected with CSRF tokens.
- All DB queries use prepared statements (PDO).
- Uploaded files are renamed to random names and the `uploads/`
  folder blocks PHP execution via `.htaccess`, even if a malicious
  file extension slips through.
- Turn `display_errors` off in `config/config.php` in production
  (already the default) — errors are logged instead of shown to visitors.
- Always serve the live site over HTTPS — required for Stripe, M-Pesa,
  and for cookies/sessions to be secure.

## 6. Nginx note

This project's `.htaccess` rules (blocking `.sql`/`.md` access,
disabling directory listing, blocking PHP execution inside `uploads/`)
are Apache-specific. If you deploy on Nginx, translate these into your
server block, e.g.:

```nginx
location ~* \.(sql|md|env)$ { deny all; }
location ^~ /uploads/ {
    location ~ \.php$ { deny all; }
}
autoindex off;
```

## 7. Customization

- Colors/branding: `assets/css/style.css` (CSS variables at the top).
- Site name, motto, mission, currency: `config/config.php`.
- Add more product categories: Admin panel inserts into `categories`
  table, or add rows directly via SQL.

---

Built as a solid starting point — review it against your host's
requirements and Stripe/Safaricom's latest API docs before going live.
