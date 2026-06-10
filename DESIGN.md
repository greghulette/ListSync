# ListSync — Design Document

A simple, fast, **shared list app** for two phones (Greg's Galaxy S25 + wife's Pixel 11).
Add items to lists, check them off, clear the checked ones, and have everything sync
automatically between both phones.

> Working name: **ListSync**. Other name ideas: OurList, DuoList, TwoCart, Nest, Stockpile, Honey-Do. Rename anytime.

---

## TL;DR (the two decisions that matter)

1. **How the phones sync** → use **Firebase** (a free Google cloud service). You will *not*
   write your own server. Firebase stores the data and pushes changes to both phones in
   real time. This decision is basically made — it's the easy, free, beginner-friendly choice.

2. **How you build the app itself** → ✅ **DECIDED: A. PWA** (web app + Firebase). Other paths, for reference:
   - **A. PWA** (a web app you "install" to the home screen) — *recommended start* ⭐
   - **B. Flutter** (a real native app, bigger learning curve) — best long-term ⭐
   - **C. Native Android** (Kotlin) — the "official" way, overkill here
   - **D. No-code** (Glide/AppSheet) — working app in an afternoon, least control

Everything else below is detail. The rest of the build flows from those two choices.

---

## 1. Goals (v1 / MVP — the first version we actually finish)

- Create multiple **named lists** (Groceries, Hardware Store, Costco, To-Do…)
- **Add an item** to a list (just a name is enough to start)
- **Check / uncheck** an item to mark it purchased
- **Clear all checked items** from a list in one tap
- **Edit / delete** items and lists
- **Sync automatically** between both phones (within a second or two)
- **Simple login** so only you two can see your lists
- **Fast & dead simple** — adding an item should take under 3 seconds

## 2. Non-goals (explicitly NOT in v1 — keeps us from drowning)

- iOS support (you're both on Android — big simplification, we lean into it)
- Barcode scanning, price tracking, recipes, meal planning
- Categories / store aisles, drag-to-reorder (maybe v2)
- Guaranteed offline editing (nice if it's free, not required)
- Sharing with lots of people / public lists
- Reminders / notifications (maybe v2)

## 3. Users

| User | Phone | OS |
|------|-------|----|
| Greg | Samsung Galaxy S25 | Android |
| Wife | Google Pixel 11 | Android |

Both Android → **no iOS needed.** This is why a web app (PWA) is so attractive here:
the usual "iPhones handle web apps poorly" problem doesn't apply to you.

## 4. Core user stories

- I open the app and see **all our lists** on the home screen.
- I tap **+ New List**, name it, and it appears for both of us.
- I tap a list to open it and see its items.
- I type an item name, hit add, and it shows up **instantly on both phones**.
- I tap a checkbox to mark something purchased — it greys out and drops to the bottom.
- I tap **Clear checked** to wipe all the purchased items at once.
- I long-press an item to **edit or delete** it.
- When my wife adds "milk" at the store, **it appears on my phone** a second later.

## 5. Screens (v1)

**Home / Lists**
```
┌─────────────────────────────┐
│  Our Lists            ⚙      │
├─────────────────────────────┤
│  🛒 Groceries          12 ▸  │
│  🔧 Hardware Store      3 ▸  │
│  📦 Costco              7 ▸  │
│                             │
│           [ + New List ]    │
└─────────────────────────────┘
```

**List detail**
```
┌─────────────────────────────┐
│  ‹ Groceries            ⋮   │
├─────────────────────────────┤
│  [ Add an item…        ] +  │
├─────────────────────────────┤
│  ☐  Milk                    │
│  ☐  Eggs                    │
│  ☐  Bread                   │
│  ───────── purchased ─────  │
│  ☑  Bananas   (greyed out)  │
│  ☑  Coffee    (greyed out)  │
│                             │
│        [ Clear checked ]    │
└─────────────────────────────┘
```

**Login** (one time): sign in with Google or email so your data is private + synced.
**Settings** (small): invite your wife, sign out.

## 6. Data model (how the info is stored)

Two main "tables" (Firebase calls them *collections*): **lists** and **items**.

```jsonc
// a list
{
  "id": "abc123",
  "name": "Groceries",
  "members": ["greg_uid", "wife_uid"],   // who can see it
  "createdBy": "greg_uid",
  "createdAt": 1733800000
}

// an item
{
  "id": "xyz789",
  "listId": "abc123",      // which list it belongs to
  "name": "Milk",
  "checked": false,
  "quantity": 1,           // optional, can add later
  "addedBy": "greg_uid",
  "createdAt": 1733800100,
  "checkedAt": null
}
```

"Clear checked" = delete every item in the list where `checked == true`.
Sync works because both phones are *watching* the same lists/items in Firebase; when one
phone changes a record, Firebase notifies the other phone automatically.

## 7. Sync & accounts — the important part

To have two phones see the same data, you need a tiny bit of cloud in the middle. You do
**not** build or run a server. You use a **Backend-as-a-Service**:

### ⭐ Recommended: Firebase (by Google) — *Cloud Firestore*
- **Real-time sync is built in.** One phone writes → the other updates automatically. This
  is the single biggest reason to use it; it's the hard part, done for you.
- **Built-in login** — including **Google sign-in**, which is perfect since you're both on
  Android with Google accounts (one tap, no passwords to manage).
- **Free tier ("Spark plan")** is *far* more than two people will ever use → **$0**.
- Works with every frontend option (PWA, Flutter, native).

### Alternative: Supabase
Open-source, uses a normal SQL database (Postgres). Also excellent. Firebase is a touch
easier for a first-timer, so we'll start there; nothing's lost if you switch later.

## 8. How to actually build the app (frontend options)

You're new and only need Android, so here are the realistic paths, **easiest first**:

### A. Progressive Web App (PWA) — HTML/CSS/JavaScript + Firebase ⭐ *recommended start*
- You build it like a web page, then "Add to Home Screen" on each phone so it opens
  full-screen and behaves like an app (its own icon, no browser bar).
- **Easiest on-ramp**, the most tutorials, and you can build & test it right in your
  computer's browser before it ever touches a phone.
- **No app store, no Android Studio, no app signing.** Host it free (Firebase Hosting,
  Netlify, or Vercel). Both phones just open the same web address → instant "deployment."
- Downside: technically not a "native" app. For a checklist app, you genuinely won't notice.

### B. Flutter (Dart language) + Firebase ⭐ *best long-term*
- One codebase → a **real installable Android app** (and iOS too, if you ever want it).
- Beautiful UI, "hot reload" (see changes instantly), and first-class Firebase support.
- Bigger learning curve: install Android Studio or VS Code, learn the Dart language.
- You can install it straight onto your two phones over USB/Wi-Fi — **no Play Store needed**
  for personal use.

### C. Native Android (Kotlin + Jetpack Compose) + Firebase
- The "official" Google way. Most powerful, **steepest curve**, and overkill for a list app.
- Worth knowing it exists; not where a beginner should start for this.

### D. No-code (Glide or Google AppSheet)
- Point it at a Google Sheet and it generates a syncing phone app in an afternoon.
- Almost **zero coding**, least control, some features need a paid plan. A good reality-check
  / fallback if the coding route ever stalls.

**Recommendation:** Start with **A (PWA + Firebase)** to get a real, syncing app on both
phones quickly and learn the moving parts (data, sync, login, UI). If you catch the bug,
graduate to **B (Flutter)** for a polished native app — the Firebase knowledge carries over.

## 9. Suggested roadmap (small, finishable milestones)

| Milestone | What "done" looks like |
|-----------|------------------------|
| **M0 — Setup** | Tools installed; a "hello world" page opens on your phone |
| **M1 — UI only** | One hard-coded list shows on screen (no cloud yet) |
| **M2 — Cloud read/write** | Hook up Firebase; one list of items reads/writes to the cloud; watch it sync between your computer and phone |
| **M3 — Core actions** | Add item / check / uncheck / clear-checked, all against the cloud |
| **M4 — Multiple lists** | Create, rename, delete lists |
| **M5 — Accounts & sharing** | Log in; your wife's phone signs in and sees the same lists |
| **M6 — Polish** | Edit/delete items, ordering, nicer look |
| **M7 — Extras (optional)** | Quantities, categories, reminders |

We do these **one at a time**. Each milestone is a thing you can show your wife and use.

## 10. Cost

- Firebase free **Spark** tier: **$0** for your usage.
- PWA hosting (Firebase/Netlify/Vercel free tiers): **$0**.
- Installing a Flutter/native app on **your own** phones: **$0**.
- Google Play Store developer account: a one-time **$25** — *only if you ever publish
  publicly*, which you don't need to (this is just for you two).
- **Total: effectively free.**

## 11. Decisions

- ✅ **Build approach:** A — **PWA** (web app: HTML/CSS/JavaScript) + Firebase. Build & test
  in the browser, then "Add to Home Screen" on both phones.
- ✅ **Greg's web experience:** some (HTML/JS, e.g. ESP32 config pages) — so we can move at a
  moderate pace, not crawl.
- ✅ **Backend / sync:** Firebase (Cloud Firestore for data + Firebase Auth for login).

Still open (decide as we go):

- **App name** — keep "ListSync" or pick something?
- **v1 items** — name-only to start, or include **quantity** right away? *(Starting name-only.)*
- **Login** — **Google sign-in** (easiest for you both) or email + password? *(Leaning Google.)*

## 12. Build progress

- **M1 — UI only, runs locally:** ✅ built → `index.html`. A complete working list app that
  saves to the browser (no cloud yet). Open it by double-clicking the file. This is the
  "see it work with zero setup" step.
- **M1.1 — Due dates + calendar reminders:** ✅ added to `index.html`. Each item can have a
  due date (📅 button); overdue items show a red chip, due-today amber. A "🔔 Remind" button
  opens Google Calendar pre-filled (no server, no Firebase needed). Picked the Google
  Calendar shortcut (tier 3) over true push for now.
- **M1.2 — Edit items + installable:** ✅ added a ✏️ edit button per item (rename via prompt),
  plus `manifest.json` + `icon.svg` so it installs to the phone home screen and runs
  full-screen. Ready to host and try on the phone (still no sync — each device is separate
  until M2).
- **M1.3 — Times on due dates:** ✅ items can have an optional time (⏰ chip); the calendar
  link becomes a timed 1-hour event with the right timezone. Direct "silent add to a chosen
  calendar" needs Google sign-in + Calendar API (see §13) — deferred to pair with M2.
- **M2 — Firebase sync:** ⬜ next. Swap the browser-storage for Firebase so both phones share
  the same data in real time.

## 13. Due dates & reminders (feasibility + plan)

**Due dates: easy, zero limitations.** A due date is just one more field on an item
(`dueDate`). HTML has a built-in date picker — `<input type="date">` — that pops the native
Android calendar. We can show due dates, sort by them, and color overdue items red. Can go
into M1 (local) right now and carries over to Firebase unchanged.

**Reminders: possible, in three tiers (easy → hard).**

1. **In-app reminders — free, easy, works on a PWA today.**
   - Overdue items turn red; group/sort by "due today / this week."
   - A count badge on the app icon (Badging API) showing how many items are due.
   - Covers most of the real-world value with almost no extra moving parts.

2. **Push notifications that buzz the phone even when the app is closed — YES on Android.**
   - PWAs *do* support push on Android (Chrome) via a service worker + the Push API + FCM
     (Firebase Cloud Messaging). You're both on Android — exactly the case where this works
     well. (iPhones are where PWA push is painful; not our problem.)
   - **The honest catch:** a PWA can't reliably schedule a "remind me next Tuesday" all by
     itself while it's closed. You need a small always-on helper in the cloud (a scheduled
     job) that checks due dates and *sends* the push. That helper = a Firebase Cloud Function
     on a timer (needs Firebase's pay-as-you-go "Blaze" plan — still ~$0 for our usage, but a
     credit card on file). Free-ish alternative: a scheduled GitHub Action that sends pushes.
   - This is the most advanced piece of the whole project → a **later milestone**, not v1.

3. **The shortcut (90% of the value, ~10% of the work): hand it to Google Calendar.**
   - When an item gets a due date, the app creates a Google Calendar event/reminder, and
     Google's own notifications do the buzzing — rock-solid, and you both already use it.
   - Very little code, no push infrastructure, no paid plan.

**Plan:** ✅ **due dates + Google Calendar reminders** landed early in M1 — they need no
backend, so there was no reason to wait. Optional later: in-app niceties (app-icon badge —
needs the installable-PWA setup) and true push (M8). For now, the Google Calendar trick is
our reminder.

**Silent "add to a specific calendar" (no popup):** the pre-filled link always shows a
confirm/Save step and lands on your *primary* calendar — that's a limit of the link trick,
not a bug. To insert events silently onto a chosen calendar (e.g. a shared "Household"
calendar) we'd use the **Google Calendar API**, which needs a one-time Google sign-in +
permission grant (client-side OAuth, no server) and a *stable* hosted URL for the OAuth
origin. Plan: bundle this with **M2** — we're adding Google sign-in there anyway, so it's one
login for both sync and calendar. On the phone, the current link is just one extra "Save" tap
(it opens the Calendar app pre-filled), so it's livable in the meantime.

---

*Doc v0.1 — starting point, expect it to change as we go.*
