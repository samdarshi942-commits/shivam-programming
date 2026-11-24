# Shivam Programing — Firebase real-time social website

This package upgrades the supplied HTML design into a Firebase-backed social app.

## Features
- Email/password sign-up and sign-in
- Google sign-in popup
- Firestore live post feed
- Create text/image posts (images stored in Firebase Storage)
- Live likes and comment creation
- User directory / Explore
- One-to-one live chat
- Editable display name and bio
- Responsive layout based on the original dark design

## 1. Create and configure Firebase
1. Open https://console.firebase.google.com/ and create a project (for example `shivam-programing`).
2. In Project settings → General → Your apps, add a Web app (`</>`).
3. Copy the Firebase config object shown by Firebase.
4. Open `index.html` and replace the six `PASTE_...` values in `firebaseConfig`.
   - `storageBucket` must match the exact bucket value shown in Firebase config. Newer projects often use `PROJECT_ID.firebasestorage.app`; older projects may show `PROJECT_ID.appspot.com`.
5. Authentication → Sign-in method: enable **Email/Password** and **Google**.
6. For Google, select a support email and save. Add your deployed domain under Authentication → Settings → Authorized domains (localhost is normally already allowed).

## 2. Enable database and storage
1. Firestore Database → Create database. Start in production mode.
2. Open Firestore → Rules, paste `firestore.rules`, and Publish.
3. Storage → Get started. Open Storage → Rules, paste `storage.rules`, and Publish.
4. If Storage asks you to attach billing / upgrade plan, follow the current Firebase console prompt. Storage availability and quotas depend on your Firebase project and plan.

## 3. Run locally
Because this uses JavaScript modules, do not open `index.html` by double-clicking it.
- VS Code: install the **Live Server** extension, right-click `index.html` → Open with Live Server.
- Or run a local static server from this folder, e.g. `python -m http.server 5500`, then open `http://localhost:5500`.
- Register two different test accounts in separate browsers/incognito windows to test chat and posts.

## 4. Deploy
You can deploy the folder to Firebase Hosting:
1. Install Node.js.
2. Install Firebase CLI: `npm install -g firebase-tools`
3. In this folder run `firebase login`
4. Run `firebase init hosting`, select your existing project, set public directory to `.` and configure as a single-page app: **No** is fine for this single HTML page.
5. Deploy with `firebase deploy --only hosting`.

Alternatively, upload the files to another static host such as Netlify or Vercel. Add that site's domain to Firebase Authentication → Authorized domains.

## Firestore data layout
- `users/{uid}` — public profile
- `posts/{postId}` — feed posts, likes array, image URL
- `posts/{postId}/comments/{commentId}` — comments
- `chats/{sortedUid1_sortedUid2}` — chat members and latest message
- `chats/{chatId}/messages/{messageId}` — message history

## Important notes / production hardening
- This is a starter implementation, not a fully moderated public social network.
- Firestore rules allow signed-in users to read the user directory and posts. Do not put private data in public user documents.
- The included UI writes likes as an array on a post. For very high-traffic posts, move likes/comments to subcollections and consider Cloud Functions.
- The app displays the live feed and chat with Firestore listeners. The simple profile `lastSeen` field is updated at sign-in; it is not a guaranteed online-presence indicator. True online presence needs Firebase Realtime Database presence + `onDisconnect`.
- The first message flow creates the chat document before writing its first message; the rules below allow the chat member to create the chat document.
- Consider App Check, rate limiting / moderation, reporting, blocking, pagination, and privacy review before a public launch.
- Never commit service-account private keys. The web app config is intended for client apps; security comes from Authentication and correctly deployed rules.

## Troubleshooting
- `auth/unauthorized-domain`: add your current host/domain to Authentication → Settings → Authorized domains.
- `Missing or insufficient permissions`: check that Firestore rules were published and you're signed in.
- Image upload fails: confirm Storage is enabled, storage rules are published, and `storageBucket` is exact.
- Google popup blocked: allow popups for your local/deployed site.
- Posts query needs an index: the current simple `createdAt` query normally does not; follow the index link in the Firebase error if Firebase reports one.


## Social features upgrade (November 2026)
This build adds:
- Follow / unfollow using `users/{uid}.following` and `users/{uid}.followers`
- Live comments displayed below posts
- Photo/video stories with a 24-hour visibility expiry (`expiresAt`)
- Video Reels feed and upload
- WebRTC voice-call offer/answer and ICE signaling stored in Firestore

### Required Firebase setup
1. Publish the updated `firestore.rules` in Firebase Console → Firestore Database → Rules.
2. Publish the updated `storage.rules` in Firebase Console → Storage → Rules.
3. Enable Authentication → Sign-in methods → Email/Password and Google.
4. Enable Firestore and Storage.
5. Open the site over HTTPS or localhost for microphone access. For reliable voice calls across different networks, configure a TURN server in the `RTCPeerConnection` `iceServers` list; the included public STUN server alone will not work on every network.
6. Configure Firebase Storage billing/limits as needed for video uploads. Story visibility is enforced by the app after 24 hours. For automatic permanent deletion, configure a Firestore TTL policy on the numeric `expiresAt` field only after converting it to a Firestore timestamp field; this build currently stores milliseconds as a number, so app visibility expiry works but Firestore TTL is not enabled.

### Important
This is a Firebase-backed starter, not a complete Instagram clone. Test calls with two signed-in devices/browsers. Voice-call signaling and media access depend on browser permissions, HTTPS, network conditions and TURN configuration. Never publish permissive production security rules.
