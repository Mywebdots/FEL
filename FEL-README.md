# FEL — Field Equipment Log
### Applied Ecology

A single-file web app for tracking field equipment deployments across projects.  
Built to answer: *What equipment is in the field right now? For which project? Since when?*

---

## What it is

A browser-based app. No server to run, no install.  
You open a URL, sign in with Google, and it's live. Everyone on the same URL sees the same data in real time.

**Hosted on:** GitHub Pages (free static hosting)  
**Data stored in:** Google Firebase Firestore (free tier, cloud database)  
**Authentication:** Google Sign-In (restricted to authorized email addresses)

---

## The whole tech stack

| Layer | What | Why |
|---|---|---|
| **App file** | One `index.html` file | No build step, no framework, just HTML + CSS + JS |
| **Hosting** | GitHub Pages | Free, deploys on commit, serves over HTTPS |
| **Database** | Firebase Firestore | Real-time sync, free tier, no server needed |
| **Authentication** | Firebase Auth (Google Sign-In) | Free, no user management overhead, permission-based access |
| **Fonts** | Google Fonts (DM Sans + DM Mono) | Loaded via CDN link in the HTML |
| **Firebase SDK** | Loaded from `gstatic.com` CDN | No npm, no bundler |

That's it. Nothing else.

---

## File location

| File | Purpose |
|---|---|
| `C:\Users\User\Downloads\index.html` | The entire app. One file. |
| Your GitHub repo `index.html` | The deployed version (must be re-uploaded after changes) |

---

## How to update the app

1. Edit `C:\Users\User\Downloads\index.html` (with Claude Code or a text editor)
2. Go to your GitHub repo → click `index.html` → upload/replace the file
3. GitHub Pages rebuilds in ~60 seconds
4. Refresh the browser — done

---

## Firebase (the database)

### Project details

| Setting | Value |
|---|---|
| **Project name** | ai-fel |
| **Project ID** | `ai-fel` |
| **Auth domain** | `ai-fel.firebaseapp.com` |
| **Storage bucket** | `ai-fel.firebasestorage.app` |
| **Console URL** | https://console.firebase.google.com → select `ai-fel` |

### API config (embedded in `index.html`)

```javascript
apiKey: "AIzaSyAsPqMNnz2NpKcRTpcazMdHc3ewvfoCacI"
authDomain: "ai-fel.firebaseapp.com"
projectId: "ai-fel"
storageBucket: "ai-fel.firebasestorage.app"
messagingSenderId: "1015418417779"
appId: "1:1015418417779:web:7f4c28305e5da220c97006"
```

### Firestore collections (tables)

| Collection | Fields | What it stores |
|---|---|---|
| `equipmentTypes` | `name`, `createdAt` | The dropdown list of equipment types (Bat Detector, Camera Trap, etc.) |
| `fieldStaff` | `name`, `createdAt` | The dropdown list of staff names |
| `equipment` | `type`, `name`, `serial`, `notes`, `createdAt` | Each physical piece of equipment |
| `projects` | `name`, `owner`, `desc`, `createdAt` | Each project |
| `logs` | `eqId`, `projId`, `person`, `dateOut`, `dateIn`, `location`, `serial`, `createdAt` | Each field deployment event |
| `services` | `eqId`, `date`, `note`, `createdAt` | Service history and maintenance notes for equipment |

### Security rules

Firestore is protected by email-based access control. Only authorized users can read or write data:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /{document=**} {
      allow read, write: if request.auth != null 
        && request.auth.token.email in [
          'mywebdots@gmail.com',
          'acarey41@gmail.com',
          'appliedecologynsw@gmail.com'
        ];
    }
  }
}
```

**To add or remove authorized users:**
1. Firebase Console → project `ai-fel` → Firestore → Rules
2. Edit the email list in the rules
3. Click Publish

**Important:** These rules require authentication. If you see permission errors, ensure:
- Your Firestore rules are set correctly (above)
- You are signed in with an authorized email address
- The Firestore listeners start ONLY after authentication is confirmed

---

## How the code is structured

The `index.html` has **three parts**:

### 1. HTML + CSS (top of file)
Standard HTML structure with inline CSS. Uses CSS variables for the green colour theme.  
Unicode characters (emoji, special symbols) are fine here.

### 2. Plain `<script>` block (middle)
All UI functions that run in the browser:
- `showTab()` — switches between the 5 tabs
- `renderEquipment()`, `renderProjects()`, `renderLogs()` — draw the tables
- `refreshTypeDropdown()`, `refreshStaffDropdown()`, etc. — populate the dropdowns
- `logOut()`, `logIn()`, `addEquipment()`, `addProject()` — handle button clicks

**CRITICAL RULE:** This block must be **100% pure ASCII characters**.  
No em-dashes (`—`), no ellipses (`…`), no box-drawing characters (`═`).  
If any non-ASCII character sneaks into this script block, the browser throws  
`SyntaxError: Unexpected token` and the entire app breaks (tabs stop working,  
dropdowns go empty). Use only plain `-` hyphens and `...` dots in comments.

### 3. `<script type="module">` block (bottom)
Firebase only. This block:
- Initializes Firebase Authentication and Firestore
- Handles Google Sign-In with email validation
- Monitors authentication state via `onAuthStateChanged`
- **Only starts Firestore listeners AFTER user is authenticated and authorized** (critical for permission rules)
- Replaces the stub functions (`_fbAddDoc`, `_fbDeleteDoc`, `_fbUpdateDoc`) with real ones
- Sets up real-time listeners (`onSnapshot`) that update the UI instantly when data changes

Module scripts are isolated — they do not pollute the global scope, which is why  
Firebase functions are explicitly assigned to `window._fbAddDoc` etc.

**Authentication flow:**
1. `onAuthStateChanged` listener fires immediately on page load
2. If no user is logged in → shows login screen
3. If user signs in → checks email against allowed list
4. If email authorized → starts all Firestore listeners, shows app
5. If email not authorized → signs out user, shows error, hides app

---

## The 6 tabs

| Tab | What it does |
|---|---|
| **Equipment** | Add physical equipment items to the register. Pick type from dropdown (or add new type inline). Each item shows current status: Available or In field. |
| **Projects** | Add projects. Use the dropdown to see existing projects before adding — prevents duplicates. Fields: name, owner, description. |
| **Log** | Deploy equipment to the field. Select equipment + project + deployed by + date + location hint. Tables below show what's currently out and full history. |
| **Reports** | Pick a project to see a summary: how many items, total deployments, what's still out, full deployment history per item. |
| **Service** | Track equipment maintenance and service history. Select equipment, record service date and notes. Edit or delete notes as needed. |
| **Lists** | Manage the dropdown lists: Equipment types and Field staff. Add or remove items here — changes appear immediately for everyone. |

---

## Setting up your data

The app starts with empty lists for equipment types and field staff. Add them manually via the **Lists tab**:

1. Go to **Lists tab**
2. Under "Equipment types" — enter type name (e.g. "Bat Detector") → click Add
3. Under "Field staff" — enter staff name (e.g. "Anne Carey") → click Add
4. Changes sync instantly to all users

You can also add items inline from the Equipment or Log tabs using the "+ Add new type..." / "+ Add new person..." options.

---

## Adding new equipment types or staff

Two ways:
1. **Lists tab** in the app → type a name → Add
2. **Equipment tab** → Equipment Type dropdown → "+ Add new type..." option
3. **Log tab** → Deployed by dropdown → "+ Add new person..." option

Changes sync instantly to all users.

---

## Deploying equipment (the main workflow)

1. Go to **Log tab**
2. Select equipment from dropdown
3. Select project from dropdown
4. Select who deployed it
5. Enter date deployed
6. Optionally enter a location hint (e.g. "Northern transect, near creek crossing")
7. Click **Deploy to field**

To record a return:
- In the **Currently in field** table → click **Return from field** → enter return date

---

## If something breaks

### Login screen shows but won't sign in
- Check your internet connection
- Ensure Google Sign-In is enabled in Firebase Console → Authentication → Sign-in method
- Try again — popup might have been blocked by browser
- Check browser console (F12) for error messages

### Sign-in says "Access denied"
Your email is not in the authorized list. Contact the app administrator to be added.

### "Missing or insufficient permissions" error on Firestore operations
Possible causes:
- Firestore rules are not set correctly. Compare with the rules in the Security rules section (above)
- You are not signed in or your session expired. Sign out and sign back in
- Firestore listeners started before authentication was confirmed. This should not happen with the current code

### Tabs not clicking / dropdowns empty
Almost certainly a non-ASCII character in the plain `<script>` block.  
Open browser DevTools (F12) → Console tab → look for `SyntaxError`.  
Fix: find and remove/replace any non-ASCII characters in the JS block.

### "Connecting..." never changes to "Live"
Firestore can't connect. Possible causes:
- Your GitHub Pages domain is not in Firebase's Authorized Domains list
- Firebase Console → Authentication → Settings → Authorized domains → add `your-username.github.io`
- Firestore security rules are blocking access. Verify the rules match the Security rules section (above)
- Firestore listeners are not starting. Check browser console for permission errors

### Data not saving ("Still connecting to database...")
Firebase module hasn't loaded yet. Wait a second and try again.  
If persistent, check Firestore rules and auth status in browser console (F12).

---

## Authentication setup

### Authorized domains

Any domain that accesses the app needs to be listed in:  
Firebase Console → project `ai-fel` → Authentication → Settings → Authorized domains

Currently should include:
- `localhost` (for local testing)
- `your-username.github.io` (for the live GitHub Pages site)

### Google Sign-In provider

Google must be enabled as a sign-in provider:
1. Firebase Console → project `ai-fel` → Authentication → Sign-in method
2. Ensure "Google" is listed and enabled
3. Ensure a Web SDK configuration is set up

### Authorized users

Edit the list of authorized emails in Firestore security rules:
- Firebase Console → project `ai-fel` → Firestore → Rules
- Update the email array in the `allow read, write:` rule
- Click Publish

Currently authorized:
- `mywebdots@gmail.com`
- `acarey41@gmail.com`
- `appliedecologynsw@gmail.com`

---

## Free tier limits (Firebase Spark plan)

| Resource | Free limit | Likely usage |
|---|---|---|
| Firestore reads | 50,000/day | Very unlikely to hit |
| Firestore writes | 20,000/day | Very unlikely to hit |
| Firestore storage | 1 GB | Tiny — text only |
| Hosting (GitHub Pages) | Unlimited | Free |

This app will almost certainly never approach these limits for a small ecology team.

---

## Quick reference — where things are

| Thing | Where |
|---|---|
| Edit the app | `C:\Users\User\Downloads\index.html` |
| Deploy changes | Upload to GitHub repo → Settings → Pages |
| View/edit database | console.firebase.google.com → ai-fel → Firestore |
| Check/edit Firestore rules | Firebase Console → ai-fel → Firestore → Rules |
| Update authorized user emails | Firebase Console → ai-fel → Firestore → Rules (edit email list) |
| Add authorized domains | Firebase Console → ai-fel → Authentication → Settings |
| Enable/disable Google Sign-In | Firebase Console → ai-fel → Authentication → Sign-in method |
| Add/remove equipment types or staff | App → Lists tab |
