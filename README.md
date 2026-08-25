# Daylesford Weekend Planner

A single-file trip planner (`index.html`, no build step) for a Daylesford weekend away. Pick activities per day, drag-reorder them into a plan, view it as a list/timeline/map, and jot notes — everything autosaves to a shared Firestore document so anyone with the link always sees the latest plan.

## Firebase setup (one-time)

The planner needs a Firebase project to persist state. Someone with a Google account does this once:

1. Create a project at [console.firebase.google.com](https://console.firebase.google.com).
2. **Build → Firestore Database → Create database**, start in production mode.
3. **Project settings → General → Your apps → Add app → Web**, register it, and copy the `firebaseConfig` object it gives you.
4. Paste those values into the `FIREBASE_CONFIG` block near the top of the `<script type="module">` in `index.html`.
5. Pick a passphrase for saving changes (anything — this just keeps random visitors from editing the plan, it's not real security). Share it with your travel partner out of band (text, verbally) — **don't commit it anywhere**.
6. In Firestore, go to **Rules** and paste in the contents of `firestore.rules` from this repo, replacing `THE_CHOSEN_PASSPHRASE` with the passphrase you picked.
7. Publish the rules, commit your filled-in `FIREBASE_CONFIG`, push to `main`.

## How persistence works

- The whole trip (selected activities, plan order, day start times, notes) lives in one Firestore document (`trips/daylesford`).
- Reads are open — the page always loads the latest saved plan, no login needed.
- Writes require the `passphrase` field to match the value in the Firestore rules. The first time you make a change, the browser prompts for the passphrase once and remembers it in `localStorage`. A wrong passphrase clears the cached value and asks again next time you edit something.
- Every change (toggling an activity, dragging to reorder, adding a custom item, changing a day's start time, editing notes) autosaves after a short debounce — no explicit "save" step.
