# Seekers Hub — setup guide

A single-file PWA (like Fit for Life) covering announcements, events, groups,
devotions with progress tracking, prayer requests, and a giving/donation page.
Built on Firebase's free Spark plan.

## 1. Create the Firebase project

1. Go to https://console.firebase.google.com → **Add project** → name it (e.g. `church-hub`).
2. Skip Google Analytics (not needed).
3. Once created, click the **web icon (</>)** to register a web app. Copy the
   `firebaseConfig` object it gives you.
4. Paste those values into `index.html`, replacing the placeholder
   `firebaseConfig` object near the top of the `<script type="module">` block.

## 2. Enable Authentication

1. In the Firebase console: **Build → Authentication → Get started**.
2. Enable the **Email/Password** sign-in method.
3. That's it — leaders and members who want personal features (devotion
   tracking, named prayer requests) will sign up right from the app.

## 3. Enable Firestore

1. **Build → Firestore Database → Create database**.
2. Start in **production mode** (the rules file below defines access, not the
   Firebase defaults).
3. Pick a region close to New Zealand (e.g. `australia-southeast1`).

## 4. Deploy the security rules

The `firestore.rules` file in this folder defines the access model:

- **Public read** on announcements, events, groups, devotions — anyone can
  view without an account.
- **Leader/admin only** can create or edit that content.
- **Prayer requests**: anyone can submit (signed in or anonymous); only
  leaders/admins can read the list.
- **Devotion progress**: private to each signed-in user.

Deploy them with the Firebase CLI:

```bash
npm install -g firebase-tools
firebase login
firebase init firestore   # point it at this folder, use the existing firestore.rules
firebase deploy --only firestore:rules
```

Or paste the contents of `firestore.rules` directly into
**Firestore → Rules** in the console and click **Publish**.

## 5. Make yourself (and other leaders) an admin

By default, every new sign-up is created with `role: "member"` — this is
enforced by the security rules so nobody can grant themselves leader access.
To promote yourself:

1. Sign up for an account in the app first (this creates your `users/{uid}` doc).
2. In the Firebase console: **Firestore Database → users → (your uid)**.
3. Edit the `role` field from `"member"` to `"admin"`.
4. Repeat with `"leader"` for other pastors/leaders who should be able to
   post announcements/events/groups/devotions and see prayer requests.

## 6. Fill in your bank details for the Give tab

Near the top of `index.html`, edit the `GIVING_DETAILS` object with your
actual account name, account number, and reference instructions.

## 7. App icons

Add two PNG icons to this folder before deploying:
- `icon-192.png` (192×192)
- `icon-512.png` (512×512)

These are referenced by `manifest.json` and used when someone installs the
app to their home screen.

## 8. Deploy hosting

**Option A — Firebase Hosting (recommended, since you're already in Firebase):**

```bash
firebase init hosting   # public directory: this folder, single-page app: No
firebase deploy --only hosting
```

**Option B — Cloudflare Pages** (same as Fit for Life): just push this folder
to a GitHub repo and connect it in Cloudflare Pages, or drag-and-drop deploy.
Either works fine — Firestore/Auth calls go straight to Firebase regardless
of where the static files are hosted.

## What's in the MVP

| Module | Who can post | Who can view |
|---|---|---|
| Announcements | Leaders/admins | Everyone |
| Events | Leaders/admins | Everyone |
| Groups | Leaders/admins | Everyone |
| Devotions | Leaders/admins | Everyone (progress tracking requires sign-in) |
| Prayer requests | Anyone (anonymous or signed in) | Leaders/admins only |
| Give | — (static bank details) | Everyone |

## Not yet included (future ideas)

- Push notifications for new announcements
- Event RSVPs
- Fit-for-life-style challenges layered on top of devotions
- Editing/deleting existing posts from the app UI (currently: Firebase console)
- Stripe/online card giving (currently: bank transfer instructions only)
