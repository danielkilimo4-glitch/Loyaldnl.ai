# Loyal DNL AI

An AI marketing tool for small businesses. Customers describe their business and get captions, a week of post ideas, ad copy, or a bio and slogans, in English, Swahili or a natural mix.

- Free plan: 3 generations per day (resets at midnight Nairobi time)
- Pro plan: KES 300 for 30 days, up to 100 generations per day, paid by M-Pesa (STK push)
- Accounts, history, an owner stats panel, and rate limiting included

## Run it on your computer

You need Node.js 20 or newer.

```bash
npm install
cp .env.example .env
```

Open `.env` and set at least:

1. `JWT_SECRET`: run `node -e "console.log(require('crypto').randomBytes(48).toString('hex'))"` and paste the result.
2. `MPESA_CALLBACK_SECRET`: generate another one the same way.
3. `ANTHROPIC_API_KEY`: from console.anthropic.com.
4. `ADMIN_EMAIL`: the email you will sign up with, so you see the owner stats panel.

Then start it:

```bash
npm start
```

Open http://localhost:3000, create an account and generate something.

### Try payments without M-Pesa credentials

Set `MPESA_MOCK=true` in `.env`. Clicking "Pay with M-Pesa" then fakes a successful payment after 5 seconds, so you can see the whole upgrade flow. This switch is ignored when `NODE_ENV=production`.

## Turn on real M-Pesa payments

1. Create an account at developer.safaricom.co.ke and make an app with the "Lipa na M-Pesa Sandbox" product.
2. Copy the Consumer Key, Consumer Secret and the test Passkey into `.env` (`MPESA_CONSUMER_KEY`, `MPESA_CONSUMER_SECRET`, `MPESA_PASSKEY`). The sandbox shortcode is `174379`.
3. Safaricom must reach your callback over public HTTPS. Set `BASE_URL` to your public address. On your own computer, use a tunnel such as ngrok and put its https URL in `BASE_URL`.
4. Test in the sandbox with the test phone number Daraja gives you.
5. To go live, apply for Go-Live on the Daraja portal with your own Paybill or Till, then set `MPESA_ENV=production`, your real shortcode and passkey, and `MPESA_TRANSACTION_TYPE` (`CustomerPayBillOnline` for Paybill, `CustomerBuyGoodsOnline` for Till; for a Till, also set `MPESA_PARTY_B` to the Till number).

If the callback does not arrive (for example on localhost), the app asks Safaricom for the payment result after 15 seconds, so upgrades still complete.

## How it is organised

| File | What it does |
| --- | --- |
| `server.js` | Routes: sign up, log in, generate, history, payments, M-Pesa callback, owner stats |
| `ai.js` | The prompts and the call to the AI. Edit this to change what each content type produces |
| `mpesa.js` | M-Pesa STK push, payment status query, phone number cleaning |
| `db.js` | SQLite database (file `data.db`) and the daily limit logic |
| `public/` | The website: `index.html`, `style.css`, `app.js` |

Prices and limits are set in `.env` (`PRO_PRICE_KES`, `PRO_DAYS`, `FREE_DAILY_LIMIT`, `PRO_DAILY_LIMIT`), not in the code.

## Before you put it online

- Set `NODE_ENV=production` and serve it over HTTPS (Render, Railway, Fly.io or a VPS with Caddy or Nginx all work). Use a host with a persistent disk, because the database is a file; back up `data.db` regularly.
- Keep `.env` private. Never commit it.
- Do a full sandbox payment test, then a real KES 300 payment from your own phone, before you tell customers.
- Add a Terms and Privacy page, since you store emails and business details.

## Ideas for next steps

- Stripe checkout for customers outside Kenya
- Password reset by email
- Save "brand profiles" so returning customers do not retype their business details
- A referral code: one free week for each friend who upgrades
