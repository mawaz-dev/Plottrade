# PlotTrade

A full-stack real estate marketplace where sellers publish property listings and buyers search, filter, and browse them. Built with React, TypeScript, Express, and MongoDB, with role-based access, Google sign-in, and image uploads.

**[Live demo →](https://plottrade.aavys.app/)**

## Features

- **Authentication**: email/password sign-up and sign-in, plus Google sign-in through Firebase
- **Buyer and seller roles**: sellers manage listings; buyers browse and contact sellers. Role checks are enforced on the server
- **Listing management**: sellers can create, edit, and delete listings and view all of their own in one place
- **Search and filtering**: text search, rent/sale type, offers, parking, furnishing, sorting, and pagination
- **Image uploads**: property photos and profile avatars are uploaded to Cloudinary
- **Profile management**: update account details and avatar, or delete the account
- **Secure sessions**: JWT stored in an HTTP-only cookie, with protected client routes and protected API routes
- **Responsive UI**: Tailwind CSS with Swiper image carousels
- **Production-ready backend**: Dockerized API that loads its secrets from AWS Secrets Manager
- **End-to-end tests**: Selenium + pytest suite using the Page Object pattern

## Tech Stack

| Layer | Technologies |
| --- | --- |
| Frontend | React 19, TypeScript, Vite, React Router, Redux Toolkit + redux-persist, Tailwind CSS 4, Swiper |
| Backend | Node.js, Express 5, TypeScript, Mongoose, JWT, bcryptjs |
| Database | MongoDB |
| Auth and media | Firebase (Google sign-in), Cloudinary (image hosting) |
| Testing | Selenium, pytest, pytest-html |
| Deployment | Docker, AWS EC2, AWS Secrets Manager, Nginx, Let's Encrypt (certbot), Vercel (client config included) |
| Tooling | pnpm |

## Architecture

```text
┌──────────────┐   /api/*    ┌───────────────┐        ┌───────────┐
│ React client │ ──────────► │ Express API   │ ─────► │  MongoDB  │
│ (Vite, 5173) │  (cookie)   │ (port 5000)   │        └───────────┘
└──────┬───────┘             └───────────────┘
       │
       ├── Firebase  → Google sign-in
       └── Cloudinary → image uploads (direct from browser)
```

- During development the Vite dev server proxies `/api` requests to `http://localhost:5000`.
- Protected API routes read a JWT from the HTTP-only `access_token` cookie. The `authorize('seller')` middleware restricts listing management to sellers.
- In production (`NODE_ENV=production`) the server fetches its configuration from AWS Secrets Manager. Locally it reads a `.env` file.

## Project Structure

```text
Plottrade/
├── client/                 # React + Vite frontend
│   ├── src/
│   │   ├── components/     # Header, Footer, FilterBar, Listing card, OAuth, ...
│   │   ├── pages/          # Home, Search, Listing, CreateListing, Profile, ...
│   │   ├── redux/          # Store and user slice (persisted)
│   │   └── App.tsx         # Routes
│   ├── firebaseConfig.ts
│   └── vercel.json         # SPA rewrite rules
├── server/                 # Express API
│   ├── controllers/        # auth, user, listing
│   ├── models/             # User, Listing (Mongoose)
│   ├── routes/
│   ├── utils/              # verifyToken, authorize, loadSecrets, errorHandler
│   ├── index.ts
│   └── Dockerfile
├── e2e/                    # Selenium + pytest end-to-end tests
├── docker-compose.yml
└── user-data-aws-lt.sh     # EC2 launch template bootstrap script
```

## Getting Started

### Prerequisites

- Node.js 22 (the Docker image uses 22; 18+ should work for development)
- [pnpm](https://pnpm.io/)
- A MongoDB database (local or [Atlas](https://www.mongodb.com/atlas))
- A Firebase project with Google sign-in enabled
- A Cloudinary account with an **unsigned upload preset**

### 1. Clone and install

```bash
git clone https://github.com/zaidy-mughal/Plottrade.git
cd Plottrade

cd server && pnpm install
cd ../client && pnpm install
```

### 2. Configure environment variables

Copy the example files and fill them in:

```bash
cp server/.example.env server/.env
cp client/.example.env client/.env
```

**`server/.env`**

| Variable | Description |
| --- | --- |
| `DB_CONNECTION_STRING` | MongoDB connection string |
| `JWT_SECRET` | Long random string used to sign JWTs |
| `PORT` | API port (defaults to `5000`) |
| `CLIENT_URL` | Allowed CORS origin(s), comma-separated, e.g. `http://localhost:5173` |
| `NODE_ENV` | `development` locally, `production` when deployed |

**`client/.env`**

| Variable | Description |
| --- | --- |
| `VITE_API_URL` | Base URL of the API. In development use `/api` (proxied by Vite) |
| `VITE_CLOUDINARY_URL` | Cloudinary upload endpoint (unsigned preset) |
| `VITE_FIREBASE_API_KEY` | Firebase web config |
| `VITE_FIREBASE_AUTH_DOMAIN` | Firebase web config |
| `VITE_FIREBASE_PROJECT_ID` | Firebase web config |
| `VITE_FIREBASE_STORAGE_BUCKET` | Firebase web config |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | Firebase web config |
| `VITE_FIREBASE_APP_ID` | Firebase web config |

> **Note:** `client/.example.env` currently lists the Firebase keys without the `VITE_` prefix, but the code reads `VITE_FIREBASE_*`. Use the prefixed names above, since Vite only exposes variables that start with `VITE_`.

### 3. Run the app

In two terminals:

```bash
# Terminal 1: API on http://localhost:5000
cd server
pnpm dev

# Terminal 2: client on http://localhost:5173
cd client
pnpm dev
```

Open <http://localhost:5173>.

## Scripts

| Location | Command | Purpose |
| --- | --- | --- |
| `client/` | `pnpm dev` | Start the Vite dev server |
| `client/` | `pnpm build` | Production build |
| `client/` | `pnpm lint` | Run ESLint |
| `server/` | `pnpm dev` | Run the API with hot reload (tsx) |
| `server/` | `pnpm build` | Compile TypeScript to `dist/` |
| `server/` | `pnpm start` | Run the compiled server |

## API Reference

All routes are prefixed with `/api`. Routes marked 🔒 require a valid auth cookie.

### Auth

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/auth/signup` | Create an account |
| `POST` | `/auth/signin` | Sign in with email, password, and role |
| `POST` | `/auth/google` | Sign in or register with Google |
| `GET` | `/auth/signout` | Clear the auth cookie |

### Listings

| Method | Endpoint | Access | Description |
| --- | --- | --- | --- |
| `GET` | `/listings` | Public | Search and filter listings |
| `GET` | `/listings/:id` | Public | Get a single listing |
| `POST` | `/listings/create` | 🔒 Seller | Create a listing |
| `PATCH` | `/listings/update/:id` | 🔒 Seller | Update a listing |
| `DELETE` | `/listings/delete/:id` | 🔒 Seller | Delete a listing |
| `GET` | `/listings/me` | 🔒 Seller | List the current seller's listings |

**Query parameters for `GET /listings`:** `searchTerm`, `type` (`sale` / `rent`), `offer`, `furnished`, `parking`, `sort` (default `createdAt`), `order` (`asc` / `desc`), `limit` (default `9`), `startIndex` (default `0`).

### Users

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/user/:id` 🔒 | Get user details |
| `POST` | `/user/update/:id` 🔒 | Update profile |
| `PUT` | `/user/update/avatar/:id` 🔒 | Update avatar |
| `DELETE` | `/user/delete/:id` 🔒 | Delete account |

### Health

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api` | API info |
| `GET` | `/health` | Health check (status, uptime, environment) |

## Data Models

**User**: `username` (unique), `email` (unique), `password` (hashed), `avatar`, `role` (`buyer` | `seller`), timestamps

**Listing**: `name`, `description`, `address`, `regularPrice`, `discountPrice`, `bedrooms`, `bathrooms`, `furnished`, `parking`, `type`, `offer`, `imageUrls[]`, `userRef`, timestamps

## Testing

The `e2e/` folder contains Selenium tests (Page Object pattern) for registration, login, and the seller listing-creation flow.

**Requirements:** Python 3, Google Chrome, and the app running locally (client on `localhost:5173`, API on `localhost:5000`).

```bash
cd e2e
pip install -r requirements.txt pillow
pytest --html=reports/report.html
```

Run the tests from inside `e2e/` because they read `test_data/users.csv` using a relative path. The login and listing tests use the accounts defined in that file, so run the registration tests first (or create those users manually).

## Deployment

**Backend (Docker + AWS)**

1. Store the server environment variables as a JSON secret in AWS Secrets Manager (default name: `prod/plottrade/mern`, region `us-east-1`). Use the same keys as `server/.env`.
2. Give the EC2 instance an IAM role that can read that secret.
3. Build and run:

   ```bash
   docker compose up --build -d
   ```

   Or launch an EC2 instance with `user-data-aws-lt.sh`, which installs Docker, pulls the image, and starts the container on port 5000.
4. Put Nginx (with a Let's Encrypt certificate) or an AWS ALB in front of port 5000.

**Frontend**

`pnpm build` in `client/` produces a static `dist/` folder. `client/vercel.json` already includes the SPA rewrite rule if you deploy to Vercel. Set `VITE_API_URL` to your public API URL and add the frontend origin to `CLIENT_URL` on the server.

## Roadmap

- Booking system for scheduling property visits and reservation requests
- Notifications for booking updates, listing changes, and account activity

## Author

**Zaidy Mughal**: [@zaidy-mughal](https://github.com/zaidy-mughal)

## License

No license has been specified yet. Add a `LICENSE` file (for example MIT) before others reuse the code.
