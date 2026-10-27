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

## How to play

- One player creates a room and shares its five-character code.
- At least three players are required; rooms support up to 12.
- Each round: answer a prompt, reframe the next player's answer, then vote for the funniest reframe.
- Players cannot vote for their own reframe. Each vote awards 100 points to the selected reframe's author.
- Five rounds, then the final leaderboard.

## Current MVP notes

- Mobile-friendly single-page HTML app; no build step is required.
- The room host advances phases via Firebase transactions when everyone has submitted.
- Anonymous sign-in is used so friends don't need accounts.
- Active game rooms are stored in Realtime Database. The MVP doesn't have a moderation queue, persistent match history, or server-side anti-cheat beyond database rules.
- Use a new room for a new match; room cleanup/expiration is a future improvement.
