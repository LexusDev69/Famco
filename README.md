# FAMCO Store - Inventory + Sales + E-Commerce (PWA)

FAMCO Store is a web-based home goods store that combines online shopping with inventory and sales management for admins and employees.

## Stack
- PHP (vanilla)
- MySQL
- HTML/CSS/JavaScript
- Bootstrap 5

## Modules
- Admin: dashboard, product CRUD, user management, inventory movements, sales management, analytics reports, activity logs
- Employee: inventory view/update, walk-in sales processing, daily/weekly/monthly report view
- Customer: storefront, search/filter, cart, checkout, order confirmation, email verification, forgot password, order tracking
- REST API:
  - `/api/get-products`
  - `/api/create-order`
  - `/api/update-stock`
  - `/api/get-reports`
- PWA: manifest, service worker, offline page

## Deploy to InfinityFree

The project includes an InfinityFree layout for the account's `htdocs` folder. The storefront opens at your domain root, and existing links under `/famco-store/public/` continue to work.

1. Create a MySQL database in the InfinityFree control panel. Copy the database hostname, database name, username, and password from **MySQL Databases**.
2. In phpMyAdmin, select the new database and import `db/schema.sql`. The SQL file creates tables in the selected database; it does not try to create a database itself.
3. Copy `config/hosting.local.example.php` to `config/hosting.local.php` and enter the database details and your HTTPS site URL. Upload `hosting.local.php` using FTP. This private file is ignored by Git and blocked from web requests.
4. Upload the website files and folders to the account's `htdocs` directory. Include `.htaccess`, `index.php`, `public`, `src`, `config`, `db`, and `PHPMailer-master`. Do not upload `.git`, Android/Gradle build files, zip archives, or any `*.local.php` files except the hosting config you just created.
5. Open `https://your-domain/famco-store/public/setup.php` once to:
   - create/reset admin account
   - auto-upgrade user table for customer email + OAuth columns
   - seed demo home catalog (Kitchen, Bedding, Bath, Dining, Home Decor, Storage)
6. Sign in at `https://your-domain/famco-store/public/login.php`, change the default admin password, then delete `public/setup.php` from the server.

InfinityFree provides the database hostname and PHP connection details in its control panel; do not use `localhost` for the hosted database. The site needs PHP 8+ with PDO MySQL enabled. Keep HTTPS enabled for PWA installation and secure session cookies. Some optional features require separate provider credentials: set `app_url` to the canonical HTTPS site URL, and configure OAuth/payment return URLs with the same domain.

### Local setup (XAMPP)
1. Create DB and tables by importing `db/schema.sql` into MySQL (select/create `famco_store` first).
2. Update DB/API settings in `config/config.php`.
3. Open `http://localhost/famco-store/public/setup.php` once to:
   - create/reset admin account
   - auto-upgrade user table for customer email + OAuth columns
   - seed demo home catalog (Kitchen, Bedding, Bath, Dining, Home Decor, Storage)
4. Login at `http://localhost/famco-store/public/login.php`.

## Customer Auth + Social Login
- Customer local auth: email/password + registration page
- Registration now requires email verification before first login
- Forgot password flow with email reset link
- Staff/Admin auth: username/password
- OAuth supported:
  - Google
  - Facebook
- To enable OAuth, set in `config/config.php`:
  - `oauth.google.enabled = true` + client keys
  - `oauth.facebook.enabled = true` + client keys
  - callback URLs must match provider app settings

## Default URLs
- Homepage: `http://localhost/famco-store/public/index.php`
- Storefront: `http://localhost/famco-store/public/shop.php`
- Main dashboard (after login): `http://localhost/famco-store/public/dashboard.php`

## Install As Mobile App (PWA)
- The app is installable on Android and iOS as a Progressive Web App.
- For install prompts, open the app on an HTTPS URL (or `http://localhost` on desktop testing).
- Android (Chrome): open site -> menu -> `Install app` / `Add to Home screen`.
- iPhone (Safari): open site -> Share -> `Add to Home Screen`.
- For local phone testing from XAMPP, use your PC LAN IP or a secure tunnel (example: ngrok) and open that URL on your phone.

## Security Implemented
- Password hashing (`password_hash`, `password_verify`)
- Session-based authentication
- Role-based access control
- API key validation via `X-API-KEY` header or `api_key` query
- Prepared statements (PDO)
- Input sanitization/validation

## SMTP / PHPMailer (Forgot Password + Verification)
Set these environment variables (example using Gmail SMTP):
- `APP_URL=http://localhost`
- `SMTP_HOST=smtp.gmail.com`
- `SMTP_PORT=587`
- `SMTP_USERNAME=your-gmail@gmail.com`
- `SMTP_PASSWORD=your-app-password`
- `SMTP_ENCRYPTION=tls`
- `SMTP_FROM_EMAIL=your-gmail@gmail.com`
- `SMTP_FROM_NAME=FAMCO Store`

Shortcut Gmail variables (auto-mapped if `SMTP_HOST` is not set):
- `GMAIL_ADDRESS=your-gmail@gmail.com`
- `GMAIL_APP_PASSWORD=your-16-char-app-password`

Install PHPMailer in the project root:
- `composer require phpmailer/phpmailer`

Or place PHPMailer source under one of these paths (already supported by loader):
- `PHPMailer-master/PHPMailer-master/src`
- `PHPMailer-master/src`
- `PHPMailer/src`

If SMTP or PHPMailer is not configured, the app still creates tokens and shows a development preview link onscreen.

## Order Logistics
- Admin page: `/admin/orders.php`
- Admin transitions: `confirmed`, `shipped`, `in_logistics`, `delivered` (with optional courier/tracking)
- Customer tracking page: `/customer/order_track.php?code=...`

## PayRex Payment Gateway
Set your PayRex credentials in `config/payment.local.php`:
- `payrexx_api_secret`: your PayRex API secret, for example an `sk_*` key
- `payrexx_api_base_url`: `https://api.payrexhq.com`
- `payrexx_instance`: optional; leave blank for PayRex HQ `sk_*` keys
- `payrexx_success_url`: `http://localhost/famco-store/public/customer/payment_return.php?status=success`
- `payrexx_failed_url`: `http://localhost/famco-store/public/customer/payment_return.php?status=failed`
- `payrexx_cancel_url`: `http://localhost/famco-store/public/customer/payment_return.php?status=cancel`

You can also use environment variables:
- `PAYREXX_INSTANCE`
- `PAYREXX_API_SECRET`
- `PAYREXX_API_BASE_URL`
- `PAYREXX_SUCCESS_URL`
- `PAYREXX_FAILED_URL`
- `PAYREXX_CANCEL_URL`

Notes:
- Checkout redirects customers to the secure PayRex hosted payment page.
- For local XAMPP testing on a real phone, use your PC LAN IP in the return URLs instead of `localhost`.
- For production, use your real HTTPS domain in all Payrexx return URLs.

## Notes
- Ensure Apache `mod_rewrite` is enabled for extensionless API routes.
- Customer/API/Employee order placement is transactional:
  - order saved
  - stock deducted
  - sales log created
  - low-stock activity logged
- New operations pages:
  - `/admin/inventory.php`
  - `/admin/sales.php`
  - `/employee/reports.php`
