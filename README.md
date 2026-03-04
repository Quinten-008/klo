# ✦ Cercle

A private social app for your inner circle — built as a single-file React web app with Firebase on the backend.

## Overview

Cercle is a mobile-first social platform designed for close friends. Instead of broadcasting to the world, everything stays within your personal circle: private messaging, shared events, photo moments, and friend-only notifications.

## Features

- **Authentication** — Email/password sign-up and login with Firebase Auth. Includes password reset flow and username uniqueness checks.
- **Friends** — Send and accept friend requests by searching usernames. See who's online.
- **Messaging** — Real-time 1-on-1 chat with friends, powered by Firestore.
- **Events** — Create events with a title, date, time, location, and description. Friends are automatically invited and can RSVP. Events display a colour-coded cover and a full guest list with attendance status.
- **Moments (Photos)** — Share photos with captions to your circle. Photos are stored as base64 in Firestore and displayed in a grid.
- **Notifications** — In-app notification feed for friend requests, event invites, RSVPs, and new messages, with an unread badge on the nav.
- **Profile** — Edit your display name, username, bio, and location. Sign out from the profile screen.

## Tech Stack

| Layer | Technology |
|---|---|
| UI | React 18 (via CDN, no build step) |
| Styling | Plain CSS with CSS custom properties |
| Backend | Firebase (Auth + Firestore) |
| Fonts | Cormorant Garamond + DM Sans (Google Fonts) |

## Getting Started

This is a **zero-build** app — no npm, no bundler, no compile step required.

1. **Clone or download** `index.html`
2. **Set up a Firebase project** at [firebase.google.com](https://firebase.google.com)
   - Enable **Authentication** (Email/Password provider)
   - Enable **Firestore Database**
3. **Add your Firebase config** — find the `firebaseConfig` object near the top of the `<script>` tag and replace the placeholder values with your own project credentials:
   ```js
   const firebaseConfig = {
     apiKey: "YOUR_API_KEY",
     authDomain: "YOUR_PROJECT.firebaseapp.com",
     projectId: "YOUR_PROJECT_ID",
     ...
   };
   ```
4. **Open `index.html`** in a browser, or deploy it to any static host (GitHub Pages, Netlify, Vercel, etc.)

## Firestore Collections

| Collection | Purpose |
|---|---|
| `users` | User profiles (displayName, username, bio, location, online status) |
| `friends` | Friend relationships and request status |
| `messages` | 1-on-1 chat messages between user pairs |
| `events` | Events created by users, including RSVP data |
| `moments` | Shared photos with captions |
| `notifications` | In-app notification records per user |

## Deployment

Since it's a single HTML file, deploying is simple:

**GitHub Pages**
```
# Place index.html at the root of your repo, then enable Pages in Settings > Pages
```

**Netlify / Vercel**
Drag and drop the file into their web dashboards — no configuration needed.

## Design

- Dark, warm aesthetic with a gold (`#c9a96e`) accent colour
- Responsive: bottom tab bar on mobile, collapsible sidebar on desktop (≥ 768px)
- Safe-area aware for modern iOS devices
- Cormorant Garamond for display text; DM Sans for UI

## License

MIT — use it however you like.
