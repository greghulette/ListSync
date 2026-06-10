# ListSync

A simple shared shopping & to-do list app for two phones. Make multiple lists, add
items, check them off, clear the checked ones, edit items, set **due dates with optional
times**, and fire off **Google Calendar reminders**. Built as a Progressive Web App (PWA),
so it installs to your phone's home screen and runs full-screen like a native app.

**Live app:** https://greghulette.github.io/ListSync/

## Status

- ✅ **M1** — works locally: lists, items, check-off, clear, edit, due dates + times, calendar reminders
- ⬜ **M2** — real-time sync across phones (Firebase) + Google sign-in (also unlocks silent calendar add)

See [DESIGN.md](DESIGN.md) for the full design and roadmap.

## Run locally

Just open `index.html` in a browser. Until M2 lands, each device stores its own data
locally (no sync yet).
