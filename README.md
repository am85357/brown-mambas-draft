# Brown Mambas Live Draft — Setup

This is a **multiplayer** GitHub Pages website. Six captains can sign in with private codes; one commissioner controls settings. Picks synchronize through Cloud Firestore in real time. **It will not connect until you configure Firebase.**

## 1. Create Firebase project

1. Go to https://console.firebase.google.com/ and create a Firebase project.
2. Add a **Web app** under Project settings. Copy the `firebaseConfig` object values into `firebase-config.js`.
3. Go to **Build > Authentication > Sign-in method**, enable **Email/Password**.
4. Under Authentication > Settings > Authorized domains, add your GitHub Pages domain (`YOURUSERNAME.github.io`).
5. Go to **Build > Firestore Database**, create a database. Select a region close to your captains.
6. Open Firestore > Rules, replace the contents with `firestore.rules` and **Publish**.

## 2. Create 7 accounts and assign roles

In Firebase Authentication > Users, choose **Add user** seven times. Use these exact email addresses (or change `LOGIN_DOMAIN` to match your chosen domain):

- `captain1@draft.example.com` ... `captain6@draft.example.com`
- `commissioner@draft.example.com`

Set a distinct strong private password for each account. Firebase Email/Password requires passwords at least six characters long; use long, random codes. Share each code privately. The domain is a login identifier and does not need to receive email, but password recovery emails won't work without a real inbox.

For each created account, copy its **User UID**. In Firestore > Data, create collection `roles`, with **document ID equal to that account's UID**. For each captain, add a **number** field `team` with value `1`, `2`, ..., `6` respectively. For commissioner, add a **boolean** field `admin` with value `true`. Never use an auto-generated role document ID. **Do not create role documents for untrusted users.**

## 3. Create initial draft document

In Firestore > Data, create collection `drafts` and document ID `main` with these fields:

- `names` (array of strings): `Team 1`, `Team 2`, `Team 3`, `Team 4`, `Team 5`, `Team 6`
- `order` (array of arrays of numbers): `[[1,2,3,4,5,6],[6,5,4,3,2,1],[1,2,3,4,5,6],[6,5,4,3,2,1],[1,2,3,4,5,6],[6,5,4,3,2,1],[1,2,3,4,5,6]]`
- `picks` (empty array)
- `pickedIds` (empty array)
- `started` (boolean): `false`
- `revision` (number): `0`

**Important:** Firestore's Console can be awkward for nested arrays. Alternatively, skip manual document creation, sign in as commissioner on the website, and click **Start draft**; the commissioner will initialize this document automatically. Then click **Pause draft** if you want to configure team names before starting.

## 4. Publish to GitHub Pages

Upload `index.html` and `firebase-config.js` to the root of a GitHub repository, then go to Settings > Pages and select Deploy from a branch, main, root. The `firestore.rules` and README are for setup and do not need to be uploaded. You can also upload the entire ZIP if you extract it first.

Use the site link on six different phones. Each captain chooses a team and enters their private code. The commissioner signs in separately. The commissioner can edit names and round orders, start/pause, undo, and reset.

## 5. Testing checklist

1. Sign in as commissioner, start the draft.
2. Sign in as Captain 1 on another device. Draft a player.
3. Check that the selection appears on both devices and Captain 2 becomes active.
4. Try drafting out of turn: it should be disabled and denied by Firestore rules.
5. Check undo, pause, and reconnect behavior.
6. Verify your Google Sheet has at least **42 distinct eligible players** and displays all categories correctly. If the sheet changes during the draft, player IDs are based on normalized names; avoid renaming already drafted players.

## Security and operational notes

- The site uses Firebase Authentication passwords as private captain codes. **Never store passwords in JavaScript, GitHub, or Firestore.**
- Firestore rules prevent captains from changing settings, drafting for another team, or taking the same ID twice. They do not independently verify that a selected player belongs to the public CSV; that would require importing the player pool into protected Firestore documents or using a trusted server function.
- The commissioner can override and undo draft actions. Keep that account private.
- Draft picks are synchronized using Firestore transactions. All 42 picks are stored in one document, suitable for this small draft.
- A Firebase project can have quotas or costs; review its pricing and set budget alerts if you enable billing.
- Browser drag-and-drop works on desktop; mobile captains should use the **Draft** button.
- This version has not been tested against your live Firebase project because no Firebase configuration or credentials have been supplied.
