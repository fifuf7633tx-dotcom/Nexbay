# Nexbay — App & Game Store

A single-page app store (Play Store / Modix Store style) built with vanilla HTML/CSS/JS, Tailwind (CDN), and Firebase (Auth + Realtime Database).

## Files
- `index.html` — the entire app (UI + logic). This is the only file you need to open it.
- `manifest.json` — PWA manifest, lets people "Add to Home Screen."
- `sw.js` — minimal service worker for an offline app shell.
- `database.rules.json` — Firebase Realtime Database security rules.

## Run it
Just open `index.html` in a browser, or serve the folder with any static host (Vercel, Netlify, Firebase Hosting, GitHub Pages). No build step.

For the PWA install prompt and service worker to work, the folder must be served over `https://` (or `localhost`) — not opened as a bare `file://` path.

## Firebase setup
The app currently points at the Firebase project from your uploaded `ik.json` (`recomail-5bd5f`), using its Realtime Database. That works out of the box, but it's the same backend as your Recomail webmail app — the `apps`, `reviews`, and `users` nodes are new and separate paths, so they won't collide with Recomail's data, but you may prefer a dedicated project for a store that's meant to scale independently.

**To use your own project instead:**
1. Firebase Console → your project → Project settings → add a **Web app** → copy the config object.
2. Replace the `firebaseConfig` object near the top of the `<script>` block in `index.html`.
3. Enable **Authentication → Email/Password** sign-in method.
4. Create a **Realtime Database** (not Firestore — this app uses RTDB) and paste `database.rules.json` into the Rules tab.
5. (Optional) Enable **Storage** if you want users to upload screenshots/logos as files instead of pasting URLs — the app already checks for Storage and won't break if it's absent.

## Data model (Realtime Database)
```
apps/{appId}: { name, tagline, category, developer, developerUid, logoUrl,
                screenshots[], downloadUrl, version, size, requirements,
                packageName, description, whatsNew, rating, ratingCount,
                downloads, verified, createdAt, updatedAt }
reviews/{appId}/{reviewId}: { uid, userName, avatar, rating, text, timestamp }
users/{uid}: { name, avatar, bookmarks: {appId:true}, uploadedApps: {appId:true} }
```

## What's implemented
- Header search with live auto-suggestions, dark/light theme toggle, auth button.
- Category nav (All / Games / Mods & Tools / Trending / Top Rated) — filters without a page reload.
- Homepage: featured slider, trending row, games row, top-rated row, full grid with infinite scroll.
- App detail (full page **and** modal, both use the same renderer): screenshots, download button with 5-second countdown, bookmark, share (native share sheet or clipboard fallback), collapsible description, info grid, star-rating + review form, live review list.
- Publish form with live card preview, writes straight to the database.
- Profile page: avatar, bookmarked apps, your published apps (with delete for your own listings), logout.
- Firebase Auth (email/password) wired to Realtime Database for profiles, bookmarks, uploads, and reviews.
- Responsive: sidebar/topbar on desktop, bottom tab bar + drawer on mobile.
- PWA manifest + service worker for installability and a cached shell.

## Notes on scope
- If the database has no apps yet, the store falls back to a small set of demo listings (drawn from your own project names — Rift Racer, Void Protocol, Recomail, OX Arena, Sriey Chat) purely so the UI isn't empty on first load. Publish one real app and the grid switches to live data.
- "Download APK" opens whatever URL you put in `downloadUrl` (a direct APK link or a Google Drive share link both work) after the countdown.
- Image/screenshot uploads are URL-based by default (paste an ImgBB, Drive, or CDN link) to avoid requiring Storage billing — swap in real `<input type="file">` + `storage.ref().put()` calls if you want direct uploads.
