# FEL — Field Equipment Log
### Applied Ecology

A single-file web app for tracking field equipment deployments across projects.  
Built to answer: *What equipment is in the field right now? For which project? Since when?*

---

## What it is

A browser-based app. No server to run, no install, no login.  
You open a URL, it's live. Everyone on the same URL sees the same data in real time.

**Hosted on:** GitHub Pages (free static hosting)  
**Data stored in:** Google Firebase Firestore (free tier, cloud database)

---

## The whole tech stack

| Layer | What | Why |
|---|---|---|
| **App file** | One `index.html` file | No build step, no framework, just HTML + CSS + JS |
| **Hosting** | GitHub Pages | Free, deploys on commit, serves over HTTPS |
| **Database** | Firebase Firestore | Real-time sync, free tier, no server needed |
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
| `logs` | `eqId`, `projId`, `person`, `dateOut`, `dateIn`, `location`, `createdAt` | Each field deployment event |

### Security rules

Currently set to **test mode** — open read/write, no login required.  
This expires approximately 30 days from when the Firebase project was created.  
When it expires, go to Firebase Console → Firestore → Rules → extend or change them.

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
- Connects to Firestore
- Replaces the stub functions (`_fbAddDoc`, `_fbDeleteDoc`, `_fbUpdateDoc`) with real ones
- Sets up real-time listeners (`onSnapshot`) that update the UI instantly when data changes
- Seeds default equipment types and staff on first load if collections are empty

Module scripts are isolated — they do not pollute the global scope, which is why  
Firebase functions are explicitly assigned to `window._fbAddDoc` etc.

---

## The 5 tabs

| Tab | What it does |
|---|---|
| **Equipment** | Add physical equipment items to the register. Pick type from dropdown (or add new type inline). Each item shows current status: Available or In field. |
| **Projects** | Add projects. Use the dropdown to see existing projects before adding — prevents duplicates. Fields: name, owner, description. |
| **Log** | Deploy equipment to the field. Select equipment + project + deployed by + date + location hint. Tables below show what's currently out and full history. |
| **Reports** | Pick a project to see a summary: how many items, total deployments, what's still out, full deployment history per item. |
| **Lists** | Manage the dropdown lists: Equipment types and Field staff. Add or remove items here — changes appear immediately for everyone. |

---

## Default seed data

On first load (when Firestore collections are empty), the app seeds these defaults:

**Equipment types:** Bat Detector, Camera Trap, Data Logger, GPS Unit, Audio Recorder, Trap, Mist Net

**Field staff:** Anne Carey, Meredith Brainwood, Caroline, Eloise

After first load, manage these lists via the **Lists tab** in the app.

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

### Tabs not clicking / dropdowns empty
Almost certainly a non-ASCII character in the plain `<script>` block.  
Open browser DevTools (F12) → Console tab → look for `SyntaxError`.  
Fix: find and remove/replace any non-ASCII characters in the JS block.

### "Connecting..." never changes to "Live"
Firebase can't connect. Possible causes:
- Your GitHub Pages domain is not in Firebase's Authorized Domains list
- Firebase Console → Authentication → Settings → Authorized domains → add `your-username.github.io`
- Firestore security rules have expired (test mode 30-day limit)

### Data not saving ("Still connecting to database...")
Firebase module hasn't loaded yet. Wait a second and try again.  
If persistent, check Firestore rules in Firebase Console.

---

## Authorised domains for Firebase

Any domain that accesses the app needs to be listed in:  
Firebase Console → project `ai-fel` → Authentication → Settings → Authorized domains

Currently should include:
- `localhost` (for local testing)
- `your-username.github.io` (for the live GitHub Pages site)

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
| Check Firestore rules | Firebase Console → ai-fel → Firestore → Rules |
| Add authorized domains | Firebase Console → ai-fel → Authentication → Settings |
| Change staff or equipment types | App → Lists tab |
