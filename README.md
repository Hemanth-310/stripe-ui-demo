# Stripe Practice - SCM Developer API Billing UI

A simple frontend demo for testing the SCM-Engine Developer API subscription and Stripe billing flow through a UI.

## Project Structure

```text
stripe-practice/
├── frontend/
│   ├── index.html
│   ├── login.html
│   ├── register.html
│   ├── onboarding.html
│   ├── success.html
│   └── cancel.html
├── .gitignore
└── README.md
```

## Local Setup

The demo requires three terminals:

1. Stripe CLI webhook listener
2. SCM-Engine backend
3. Stripe Practice frontend

### Terminal 1 - Stripe Webhook Listener

Start the Stripe CLI listener:

```bash
stripe listen \
  --events checkout.session.completed,payment_intent.succeeded \
  --forward-to localhost:8000/api/payment/webhook
```

The Stripe CLI will display a webhook signing secret:

```text
Ready! You are using Stripe API Version [....].
Your webhook signing secret is whsec_...
```

Copy the `whsec_...` value and update the SCM-Engine `.env` file:

```env
STRIPE_WEBHOOK_SECRET=<STRIPE_WEBHOOK_SECRET>
```

Example:

```env
STRIPE_WEBHOOK_SECRET=whsec_...
```

Keep the Stripe CLI running in this terminal.


---

### Terminal 2 - Start SCM-Engine Backend

Open a second terminal:

```bash
cd /Users/hemanth/spectraone/SCM-Engine
```

Activate the virtual environment:

```bash
source venv/bin/activate
```

Start the FastAPI backend:

```bash
uvicorn app.main:app --port 8000 --reload
```

The backend will run at:

```text
http://127.0.0.1:8000
```

Check the backend health:

```bash
curl http://127.0.0.1:8000/health
```

Expected response:

```json
{
  "status": "ok"
}
```

Keep the backend running.

---

### Terminal 3 - Start Stripe Practice Frontend

Open a third terminal:

```bash
cd /Users/hemanth/spectraone/stripe-practice
```

Start the frontend:

```bash
python3 -m http.server 5500 --directory frontend
```

The frontend will run at:

```text
http://localhost:5500
```

Open the login page:

```text
http://localhost:5500/login.html
```

---

## Application Flow

### Login and Profile Onboarding

When a user logs in, the frontend checks whether the user's profile is complete.

#### Existing user with completed profile

```text
Login
  ↓
Check Profile
  ↓
Profile Complete
  ↓
Developer API Dashboard
```

#### New user or incomplete profile

```text
Login
  ↓
Check Profile
  ↓
Profile Incomplete
  ↓
Onboarding
  ↓
Update Profile
  ↓
Developer API Dashboard
```

The onboarding page collects:

- First Name
- Last Name
- Email Address
- Phone Number
- Country Code

The profile is updated using the SCM-Engine profile API.

---

## Developer API Billing Flow

After completing login and onboarding:

```text
Login/Signup
   ↓
Profile Check
   ↓
Developer API Dashboard
   ↓
Select Developer API Plan
   ↓
Create Payment Intent
   ↓
Stripe Payment
   ↓
Stripe Webhook
   ↓
SCM-Engine
   ↓
Subscription/Billing Updated
   ↓
Payment Success Page
```

### Create Payment Intent

The frontend calls:

```text
POST /api/developer/billing/create-intent
```

The backend creates the Stripe billing/subscription flow and returns the required PaymentIntent information to the frontend.

### Stripe Payment

The frontend uses Stripe.js / Stripe Elements to complete the payment.

For local testing, use Stripe test payment methods.

### Stripe Webhook

Stripe sends payment events to:

```text
POST /api/payment/webhook
```

The Stripe CLI forwards the events to the local SCM-Engine backend:

```text
Stripe
   ↓
Stripe CLI
   ↓
localhost:8000/api/payment/webhook
   ↓
SCM-Engine
```

The Stripe listener is configured for:

```text
checkout.session.completed
payment_intent.succeeded
```

### Payment Success

After successful payment, the frontend redirects to:

```text
success.html
```

The Developer API subscription and billing information can then be checked from the dashboard.

---

## Important URLs

| Component | URL |
|---|---|
| Frontend | http://localhost:5500 |
| Login | http://localhost:5500/login.html |
| Backend | http://127.0.0.1:8000 |
| Backend Health | http://127.0.0.1:8000/health |
| API Documentation | http://127.0.0.1:8000/docs |

---

## Stripe Configuration

The SCM-Engine backend requires the Stripe configuration in its `.env` file.

Example:

```env
STRIPE_SECRET_KEY=<STRIPE_SECRET_KEY>
STRIPE_PUBLISHABLE_KEY=<STRIPE_PUBLISHABLE_KEY>
STRIPE_WEBHOOK_SECRET=<STRIPE_WEBHOOK_SECRET>
```

The webhook signing secret is provided by:

```bash
stripe listen
```



---

## Troubleshooting

### Stripe webhook is not reaching the backend

Make sure Terminal 1 is running:

```bash
stripe listen \
  --events checkout.session.completed,payment_intent.succeeded \
  --forward-to localhost:8000/api/payment/webhook
```

Make sure the webhook secret from the current Stripe CLI session is configured in the SCM-Engine `.env` file.

### Backend is not running

Run:

```bash
curl http://127.0.0.1:8000/health
```

Expected:

```json
{"status":"ok"}
```

If the backend is not running:

```bash
cd /Users/hemanth/spectraone/SCM-Engine
source venv/bin/activate
uvicorn app.main:app --port 8000 --reload
```

### Frontend is not loading

Make sure Terminal 3 is running:

```bash
cd /Users/hemanth/spectraone/stripe-practice
python3 -m http.server 5500 --directory frontend
```

Then open:

```text
http://localhost:5500/index.html
```

### Stripe CLI is connected to the wrong Stripe account

Check the active Stripe CLI account:

```bash
stripe switch
```

Make sure the selected Stripe account matches the Stripe account configured in SCM-Engine.

---

```

Use placeholders in documentation:

```text
<STRIPE_SECRET_KEY>
<STRIPE_PUBLISHABLE_KEY>
<STRIPE_WEBHOOK_SECRET>
```



---
