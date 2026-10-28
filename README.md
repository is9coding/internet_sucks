# Internet Sucks

A mobile-first, browser-based multiplayer party game inspired by the general idea of taking answers out of context. It uses original prompts and its own interface.

## Play online

After the GitHub Pages deployment finishes, the site should be available at:

https://is9coding.github.io/internet_sucks/

## One-time Firebase setup (required for multiplayer)

The static website is hosted by GitHub Pages. Firebase Realtime Database and anonymous Firebase Authentication provide live room state and player identity. GitHub Pages alone cannot synchronize player data between phones.

1. Open the Firebase project associated with this app in the Firebase console.
2. In **Authentication → Sign-in method**, enable **Anonymous**.
3. Ensure **Realtime Database** exists in the region used by the configured database URL.
4. In **Realtime Database → Rules**, publish the contents of `database.rules.json`. You can also install the Firebase CLI and run `firebase deploy --only database` from this directory after logging in to the correct Firebase project.
5. In the GitHub repository, open **Settings → Pages** and select **GitHub Actions** as the build and deployment source if it isn't already selected.
6. Open **Actions** and confirm that **Deploy Internet Sucks to GitHub Pages** finishes successfully.

The Firebase web config in `index.html` is a client identifier, not an admin credential. The database rules are important; do not use unrestricted public read/write rules.

### If room creation says "Permission denied"

Publish the **latest** `database.rules.json` from this repository in Firebase Console → Realtime Database → Rules. The room-creation transaction must be allowed to read an unused room-code path before it can create the room. The current room-level read rule therefore allows authenticated reads when the room does not exist, while existing rooms remain readable only to their players.

Also confirm Firebase Authentication → Sign-in method → **Anonymous** is enabled. Changes to this repository's rules file do not automatically publish the rules to Firebase; you must press **Publish** in the Firebase Console (or deploy with the Firebase CLI).

## Round formats

A match has four rounds. Rounds 1–3 randomly select three distinct themes from the nine text-based website parodies: Video Site, Social Media, Social Network, Web Forum, Job Network, Store, Crowdfunding, Check-in Site, and News Site. Round 4 is always the Photo Sharing Site finale.

In rounds 1–3, each player writes the role shown for that theme. The assigned twister sees that text and answers the theme's framing prompt. The room votes on the combined entry.

In Round 4, each player is shown two photos and picks one for their personal-feed post. The assigned twister writes a caption for that photo. Voting cards and results show the photo with the caption underneath.

The photo cards are remote Unsplash images, so the finale needs an internet connection to display them.

## How to play

- One player creates a room and shares its five-character code.
- At least three players are required; rooms support up to 12.
- Each round: answer a prompt, reframe the next player's answer, then vote for the funniest reframe.
- Players cannot vote for their own reframe. Each vote awards 100 points to the selected reframe's author.
- Four rounds, then the final leaderboard.

### Updated rules after the photo finale change

The current `database.rules.json` now permits a final-round answer containing a `photoId` instead of text. Publish the latest file in Firebase Console → Realtime Database → Rules after updating the code. A GitHub commit does not automatically publish Realtime Database rules.

## Current MVP notes

- Mobile-friendly single-page HTML app; no build step is required.
- The host controls timeout outcomes: wait 60 seconds, skip one missing action, or remove that player from the current match. There is no automatic skip.
- Answer and reframe phases default to 120 seconds; voting defaults to 60 seconds. The timer is stored in shared room state and can be extended by the host.
- When the original host disconnects, active phases pause. The original host must reconnect with the same anonymous Firebase identity to resume; hosting is not transferred.
- Players can leave a room. Removed/departed players' current-round records are preserved, unfinished work is resolved as a placeholder/skip/abstention, and they are excluded from future rounds until they rejoin the lobby.
- Phase transitions and scoring are applied in Firebase transactions to avoid duplicate score awards. Database Rules reject duplicate submissions, late client submissions, votes for skipped entries, and submissions while the host is offline.
- The app is still a static browser client, not a trusted game server. Firebase Rules restrict client writes, but a full adversarial anti-cheat/authoritative scheduler would require trusted server-side functions.
- Rooms still need host/participant activity to progress and do not yet have server-side automatic expiry/cleanup.
