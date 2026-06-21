# friends-with-bikes-backend
friends with bikes backend


# Friends With Bikes — Backend

GraphQL API for [Friends With Bikes](https://github.com/lorynmason/friends-with-bikes), a social cycling app focused on exploration and storytelling rather than performance.

Built with **FastAPI**, **Strawberry GraphQL**, and **PostgreSQL + PostGIS**.

---

## Tech Stack

| Layer | Technology |
|---|---|
| API framework | FastAPI |
| GraphQL | Strawberry |
| Database | PostgreSQL + PostGIS |
| Auth | JWT via bcrypt |
| File storage | Cloudflare R2 |
| Runtime | Python 3.12 |
| Package manager | uv |

---

## Getting Started

### Prerequisites

- Python 3.12
- [uv](https://github.com/astral-sh/uv)
- PostgreSQL with PostGIS extension

### Installation

```bash
# Clone the repo
git clone https://github.com/lorynmason/friends-with-bikes-backend.git
cd friends-with-bikes-backend

# Install dependencies
uv sync

# Copy environment variables
cp .env.example .env
```

Fill in your `.env` file (see [Environment Variables](#environment-variables) below), then:

```bash
# Run database migrations
uv run alembic upgrade head

# Start the dev server
uv run fastapi dev main.py
```

The GraphQL explorer will be available at `http://localhost:8000/graphql`.

---

## GraphQL Schema

### Queries

| Query | Description |
|---|---|
| `me` | Returns the current authenticated user |
| `user(id)` | Returns a user by ID |
| `ride(id)` | Returns a ride by ID |
| `rides(userId)` | Returns all rides for a user |
| `feed(cursor, limit)` | Returns paginated rides from followed users |
| `ridesNear(lat, lng, radiusKm)` | Returns rides near a location using PostGIS |

### Mutations

| Mutation | Description |
|---|---|
| `signUp` | Create a new account |
| `login` | Authenticate and return a JWT |
| `updateProfile` | Update the current user's profile |
| `importStravaActivities` | Fetch recent activities from Strava |
| `publishRide` | Publish an imported activity as a ride post |
| `updateRide` | Edit a ride post |
| `deleteRide` | Delete a ride post |
| `followUser` | Follow a user |
| `unfollowUser` | Unfollow a user |
| `likeRide` | Like a ride |
| `unlikeRide` | Unlike a ride |
| `addComment` | Add a comment to a ride |
| `deleteComment` | Delete a comment |
| `saveRide` | Bookmark a ride |
| `unsaveRide` | Remove a bookmarked ride |

---

## Architecture

### Geospatial Search

Routes are stored with a PostGIS `geography` column on each ride's start point. The `ridesNear` query uses `ST_DWithin` to efficiently find rides within a given radius — enabling the route discovery map on the frontend.

### Strava Integration

Strava's OAuth2 redirect flow is handled by two lightweight REST endpoints (`GET /strava/connect` and `GET /strava/callback`). After authorization, all Strava data fetching is triggered via GraphQL mutations. Access tokens are refreshed automatically on expiry.

### N+1 Prevention

Strawberry's built-in DataLoader support is used for batching relational queries (followers, likes, comments) to avoid N+1 problems on the feed.

### File Uploads

Photo uploads use a dedicated `POST /upload` REST endpoint that writes to Cloudflare R2 and returns a URL. That URL is then passed into GraphQL mutations — keeping binary data out of the GraphQL layer.

---

## Environment Variables

```env
DATABASE_URL=postgresql://user:password@localhost:5432/friendswithbikes
JWT_SECRET=your-secret-key
STRAVA_CLIENT_ID=your-strava-client-id
STRAVA_CLIENT_SECRET=your-strava-client-secret
STRAVA_REDIRECT_URI=http://localhost:8000/strava/callback
R2_ACCOUNT_ID=your-cloudflare-account-id
R2_ACCESS_KEY=your-r2-access-key
R2_SECRET_KEY=your-r2-secret-key
R2_BUCKET=friendswithbikes
```

---

## Frontend

The frontend repo lives at [friends-with-bikes](https://github.com/lorynmason/friends-with-bikes) and is built with Next.js and Apollo Client.

---

## License

MIT
