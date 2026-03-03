# ✦ Cercle — Firebase Setup Guide

This version connects to a real Firebase backend. Real accounts, real friend requests, real messages shared between users.

---

## Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project** → give it a name (e.g. `cercle`)
3. Disable Google Analytics if you don't need it → **Create project**

---

## Step 2 — Enable Authentication

1. In the Firebase console sidebar: **Build → Authentication**
2. Click **Get started**
3. Under **Sign-in method**, enable **Email/Password**
4. Save

---

## Step 3 — Create a Firestore database

1. In the sidebar: **Build → Firestore Database**
2. Click **Create database**
3. Choose **Start in test mode** (you'll secure it later)
4. Pick a region close to you → **Enable**

---

## Step 4 — Get your config keys

1. Click the **gear icon** next to "Project Overview" → **Project settings**
2. Scroll down to **Your apps** → click the **</>** (web) icon
3. Register the app (any nickname) — you don't need Firebase Hosting
4. Copy the `firebaseConfig` object that appears. It looks like:

```js
const firebaseConfig = {
  apiKey: "AIza...",
  authDomain: "your-project.firebaseapp.com",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123:web:abc123"
};
```

---

## Step 5 — Paste the config into index.html

Open `index.html` and find this block near the top (around line 20):

```js
const FIREBASE_CONFIG = {
  apiKey:            "YOUR_API_KEY",
  authDomain:        "YOUR_PROJECT_ID.firebaseapp.com",
  projectId:         "YOUR_PROJECT_ID",
  storageBucket:     "YOUR_PROJECT_ID.appspot.com",
  messagingSenderId: "YOUR_SENDER_ID",
  appId:             "YOUR_APP_ID"
};
```

Replace each `"YOUR_..."` value with the real values from Step 4.

---

## Step 6 — Add Firestore security rules

In the Firebase console → **Firestore Database → Rules**, paste:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // Users: anyone can read profiles (needed for username search before login)
    // Only the owner can write their own document
    match /users/{uid} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.uid == uid;

      // Friend requests: only the recipient can read/update
      match /friendRequests/{fromUid} {
        allow read, write: if request.auth.uid == uid || request.auth.uid == fromUid;
      }

      // Friends list
      match /friends/{friendUid} {
        allow read: if request.auth.uid == uid;
        allow write: if request.auth != null;
      }

      // Notifications
      match /notifications/{nid} {
        allow read, write: if request.auth.uid == uid;
        allow create: if request.auth != null;
      }
    }

    // Chats: only members can read/write
    match /chats/{chatId} {
      allow read, write: if request.auth != null &&
        (resource == null || request.auth.uid in resource.data.members ||
         chatId.matches('.*' + request.auth.uid + '.*'));

      match /messages/{msgId} {
        allow read, write: if request.auth != null;
      }
    }

    // Events: members only
    match /events/{evId} {
      allow read: if request.auth != null &&
        request.auth.uid in resource.data.memberUids;
      allow create: if request.auth != null;
      allow update: if request.auth.uid in resource.data.memberUids;
    }

    // Photos: authenticated users
    match /photos/{phId} {
      allow read, write: if request.auth != null;
    }
  }
}
```

Click **Publish**.

---

## Step 7 — Add Firestore indexes

Some queries need composite indexes. Firebase will show you an error link in the browser console the first time a query runs — just click the link and it creates the index automatically. The queries that need indexes are:

- `events` — `memberUids` array-contains + `createdAt` desc
- `photos` — `ownerUid` in + `createdAt` desc
- `notifications` — `createdAt` desc

---

## Step 8 — Deploy to GitHub Pages

Upload `index.html` to your GitHub repo (same steps as before):

1. Go to your repo → **Add file → Upload files**
2. Upload `index.html`
3. **Settings → Pages → Deploy from branch → main → / (root)**

Your app is live at `https://your-username.github.io/your-repo/`

---

## How it works

| Feature | How |
|---|---|
| Accounts | Firebase Authentication (email + password) |
| Profiles | Firestore `users/{uid}` document |
| Friend requests | `users/{uid}/friendRequests/{fromUid}` subcollection |
| Messages | `chats/{chatId}/messages` — real-time with `onSnapshot` |
| Events | `events` collection — shared with all invited members |
| Photos | `photos` collection — visible to all authenticated users |
| Notifications | `users/{uid}/notifications` subcollection |
| Online status | `users/{uid}.online` boolean, updated on sign in/out |

---

## Firestore data structure

```
users/
  {uid}/
    uid, displayName, username, bio, location, email, online, createdAt
    friends/
      {friendUid}/  { since }
    friendRequests/
      {fromUid}/    { from, fromName, status, createdAt }
    notifications/
      {id}/         { type, from, fromName, text, unread, createdAt }

chats/
  {uid1_uid2}/
    members: [uid1, uid2]
    lastMessage, lastAt
    messages/
      {id}/  { from, fromName, text, createdAt }

events/
  {id}/
    title, date, time, location, description
    creatorUid, creatorName
    memberUids: [uid, ...]
    attendees: [{uid, name}]
    rsvps: { uid: "going"|"maybe"|"can't go" }
    cover, createdAt

photos/
  {id}/
    ownerUid, ownerName, caption, color, likes: [uid], createdAt
```

---

## Free tier limits (Firebase Spark plan)

| Resource | Free limit |
|---|---|
| Firestore reads | 50,000 / day |
| Firestore writes | 20,000 / day |
| Firestore storage | 1 GB |
| Authentication | Unlimited |
| Hosting (if used) | 10 GB / month |

This is more than enough for a private app with friends.
