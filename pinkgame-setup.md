# PinkGame Firebase setup

Use Firebase Authentication for passwords. Do **not** store passwords in Firestore.

## 1) Create the Firebase project
1. Create a new Firebase project.
2. Add a **Web app**.
3. Copy the web config into `pinkgame-admin.html`.

## 2) Turn on sign-in
Enable **Authentication → Email/Password**.

## 3) Create Firestore
Create Firestore and add these collections:

### `games/{gameId}`
```json
{
  "name": "Doom 1+2!",
  "emoji": "💥",
  "img": "https://example.com/thumb.png",
  "bg": "#111111",
  "cat": "action",
  "url": "https://example.com/game",
  "isNew": false,
  "isPop": false,
  "isSfc": true,
  "sortOrder": 2
}
```

### `profiles/{uid}`
```json
{
  "email": "you@example.com",
  "displayName": "PinkGame Admin",
  "photoURL": "",
  "role": "admin"
}
```

## 4) Make your account admin
1. Sign up in `pinkgame-admin.html`.
2. Open Firestore.
3. Find `profiles/{yourUid}`.
4. Change `role` from `member` to `admin`.

## 5) Security rules
Paste this into Firestore Rules:

```js
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    function isSignedIn() {
      return request.auth != null;
    }

    function isSelf(uid) {
      return isSignedIn() && request.auth.uid == uid;
    }

    function isAdmin() {
      return isSignedIn() &&
        get(/databases/$(database)/documents/profiles/$(request.auth.uid)).data.role == "admin";
    }

    match /games/{gameId} {
      allow read: if true;
      allow write: if isAdmin();
    }

    match /profiles/{uid} {
      allow read: if isSelf(uid) || isAdmin();
      allow create: if isSelf(uid);
      allow update: if isSelf(uid) && request.resource.data.role == resource.data.role;
      allow delete: if isAdmin();
    }
  }
}
```

## 6) Main site change
Your current `games.html` is still hardcoded. To make admin edits appear live, replace the `const games = [...]` block with a Firestore listener that reads the `games` collection.

The easiest path is to use the live `pinkgame-games.html` version below as your main page.

## 7) Passwords
Passwords are handled by Firebase Auth. That means:
- no manual hashing in your code
- no password docs in Firestore
- built-in sign-in, sign-up, and session handling

## 8) Workflow
- Add or edit games in the admin panel.
- Set `sortOrder` to control order.
- Set `isSfc = true` for the blue tag.
- The main page reads the same collection and updates automatically.
