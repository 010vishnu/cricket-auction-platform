# Premier Cricket Auction — Real-Time Auction Web App

A full-stack, real-time cricket auction platform (IPL-style) built with React + Vite,
Node/Express, MongoDB, and Socket.IO — dark/gold/orange "premium auction" theme,
built for desktop, mobile, and projector/TV display.

## Stack

- **Frontend:** React 18, Vite, Tailwind CSS, React Router, Socket.IO client, Recharts, canvas-confetti
- **Backend:** Node.js, Express, MongoDB (Mongoose), Socket.IO, JWT auth, bcrypt, ExcelJS, Multer
- **Real-time:** Socket.IO broadcasts every bid, sale, and status change to all connected admin/team/public screens instantly

## Project Structure

```
cricket-auction/
├── backend/
│   ├── config/db.js              # MongoDB connection
│   ├── models/                   # User, Team, Player, AuctionState, BidHistory
│   ├── middleware/                # auth (JWT), upload (multer), error handling
│   ├── controllers/               # business logic per resource
│   ├── routes/                    # REST API endpoints
│   ├── socket/socketHandler.js    # Socket.IO connection handling
│   ├── utils/                     # Excel import/export helpers, JWT signing
│   ├── scripts/seed.js            # sample data generator (8 teams, 120 players)
│   └── server.js                  # app entry point
└── frontend/
    └── src/
        ├── pages/                 # Home, Login, AdminDashboard, TeamDashboard, LiveAuction, Statistics
        ├── components/            # Navbar, PlayerCard, CountdownTimer, sound.js, admin/* tabs
        ├── context/                # AuthContext, SocketContext
        └── api/axios.js           # API client with JWT auto-attach
```

## Prerequisites

- Node.js 18+
- MongoDB running locally (or a MongoDB Atlas connection string)
- npm

## Setup & Run (local, VS Code)

### 1. Backend

```bash
cd backend
cp .env.example .env
# edit .env if needed (Mongo URI, JWT secret, admin credentials)
npm install
npm run seed     # creates admin user, 8 teams (with team logins), 120 sample players
npm run dev       # starts the API + Socket.IO server on http://localhost:5000
```

### 2. Frontend

Open a second terminal:

```bash
cd frontend
cp .env.example .env
npm install
npm run dev       # starts the Vite dev server on http://localhost:5173
```

Visit **http://localhost:5173**.

### Demo Logins (after running `npm run seed`)

- **Admin:** `admin` / `admin123`
- **Team:** each team's short name lowercase (e.g. `mt`, `ck`, `bw`, `dd`, `kr`, `ps`, `rr`, `hs`) / `team123`
- **Public display:** no login needed — just open `/live`

## How the Auction Flow Works

1. Admin logs in, goes to **Admin → Players / Teams** to review or edit the seeded data (or imports players via Excel — columns: `Name, Age, Role, BattingStyle, BowlingStyle, State, PreviousTeam, BasePrice`).
2. Admin opens **Live Auction**, clicks **Start Auction**, then selects a player from the dropdown to put them up for bidding.
3. Team owners (logged in) see a **Bid** button on the same `/live` page and click to bid the current increment.
4. Every bid instantly updates the price, countdown timer, and highest-bidder on **every connected screen** (admin, team, and the public/projector view) via Socket.IO — no refresh needed.
5. Admin clicks **Mark Sold** (deducts purse, adds player to the team) or **Mark Unsold**. **Undo Last Bid** reverts a mistaken bid.
6. The **Statistics** page and team dashboards update automatically with purse spent, squad size, and auction-wide analytics/charts.

## Key API Endpoints

| Method | Endpoint | Access |
|---|---|---|
| POST | `/api/auth/login` | public |
| GET/POST/PUT/DELETE | `/api/teams` | public read / admin write |
| GET/POST/PUT/DELETE | `/api/players` | public read / admin write |
| POST | `/api/players/import` | admin (multipart Excel) |
| GET | `/api/players/export/list` | admin (downloads .xlsx) |
| GET | `/api/auction/state` | public |
| POST | `/api/auction/start` \| `/pause` \| `/resume` \| `/end` | admin |
| POST | `/api/auction/select-player` \| `/sold` \| `/unsold` \| `/undo` | admin |
| POST | `/api/auction/bid` | team |
| GET | `/api/stats` | public |

## Notes on What's Included vs. Left as an Extension

Everything needed to actually run a live auction end-to-end is implemented and working:
role-based auth, team/player CRUD with search/filter/pagination, Excel import/export,
real-time bidding with Socket.IO, countdown timer, sold/unsold/undo, purse deduction,
live leaderboard, bid history, statistics with charts, sound effects (Web Audio, no
external files needed), confetti on big sales, and glassmorphism dark/gold/orange theme.

A few purely cosmetic extras from the original spec were intentionally left as easy
follow-ups so the core app stays easy to read and extend:
- Drag-and-drop player reordering in the queue (auctionOrder field already exists on the Player model — just needs a drag UI, e.g. `@dnd-kit/core`)
- Dedicated print-friendly stylesheet for auction reports (the Excel export already covers exporting results)
- Keyboard shortcuts for admin actions (Sold/Unsold/Undo are already simple button handlers you can bind to keys)

## Deployment Notes

- Set `MONGO_URI` to your production MongoDB (e.g. Atlas), set a strong `JWT_SECRET`, and set `CLIENT_URL` to your deployed frontend origin (for CORS + Socket.IO).
- Build the frontend with `npm run build` (outputs to `frontend/dist`) and serve it from any static host, or have Express serve it directly.
- Run `npm run seed` once against your production database to create the initial admin account, then change the password.
