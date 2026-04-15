================================================================================
  HYDERI SPORTS — USER GUIDE
  Tournament Management Platform
  Built by Rezahusein Karim
================================================================================

CONTENTS
--------
  1.  What Is Hyderi Sports?
  2.  Getting Started (First Run)
  3.  Admin Panel — Overview
  4.  Creating a Tournament
  5.  Managing Teams
  6.  Groups & Fixtures
  7.  Scoring Matches
  8.  Auto-Scheduling
  9.  Knockout Mode
  10. Leagues
  11. Players & Leaderboards
  12. Media (Photos & Videos)
  13. Public Site — What Spectators See
  14. Favourites & Notifications
  15. HTTPS Setup
  16. GitHub Sync
  17. Configuration Reference
  18. File Structure
  19. Troubleshooting


================================================================================
  1. WHAT IS HYDERI SPORTS?
================================================================================

Hyderi Sports is a self-hosted sports tournament management system. It runs
entirely on your local machine or Raspberry Pi with a single command — no
database, no cloud services, no subscription.

  ADMIN PANEL  ->  /admin
    Password-protected. You manage everything here: create tournaments, register
    teams, enter scores, launch knockout brackets, upload photos and videos.

  PUBLIC SITE  ->  /  (the home page)
    Open to everyone. Spectators and players view live standings, fixtures,
    knockout brackets, and the media gallery. They can follow teams and get
    notifications before matches.

Everything saves automatically to JSON files in the data/ folder. There is
nothing to configure before starting — just run the server and open a browser.


================================================================================
  2. GETTING STARTED (FIRST RUN)
================================================================================

REQUIREMENTS
  - Node.js (any modern version — v18 or later recommended)
  - A Raspberry Pi, laptop, or any Linux/Windows/Mac machine

STARTING THE SERVER

  node server.js

  The terminal will show something like:

    Hyderi Sports -- http://localhost:3000
       Admin:    http://localhost:3000/admin
       No ssl/cert.pem -- running plain HTTP
       Password: HyderiAdmin321 (default)

  Open your browser to http://localhost:3000 to see the public site.
  Open http://localhost:3000/admin to access the admin panel.

DEFAULT PASSWORD
  HyderiAdmin321

  Change it by setting an environment variable before starting:
    ADMIN_PASSWORD=YourNewPassword node server.js

  Or set it permanently in a startup script.

KEEPING IT RUNNING (Raspberry Pi)
  Use tmux so the server stays alive when you close the terminal:

    tmux new -s hyderi
    node server.js
    (press Ctrl+B then D to detach -- server keeps running)

  To come back:
    tmux attach -t hyderi


================================================================================
  3. ADMIN PANEL — OVERVIEW
================================================================================

After logging in you see the Dashboard, which shows totals for tournaments,
leagues, players, and live/completed counts.

LEFT SIDEBAR
  - List of all tournaments (click any to open it)
  - + New Tournament button
  - Leagues section
  - Players section

TOURNAMENT EDITOR (when a tournament is open)
  Four tabs at the top:

    Teams & Groups   -- register teams, assign to groups, drag and drop
    Fixtures         -- view standings, enter scores, add matches
    Knockout         -- launch and manage the knockout bracket
    Media            -- upload photos and videos

  Status buttons let you instantly switch a tournament between:
    Upcoming -> Live -> Completed

LIVE SYNC
  The admin panel polls the server every 3 seconds. If someone else makes a
  change (e.g. a score is entered on another device), your panel updates
  automatically.


================================================================================
  4. CREATING A TOURNAMENT
================================================================================

Click "+ New Tournament" in the sidebar.

Fill in:
  Name           -- e.g. "Summer Cup 2025"
  Sport / Game   -- e.g. "Football", "Basketball", "Cricket"
  League         -- optional: attach to a league for cross-tournament standings
  Start Date / End Date
  Description    -- shown on the public card

SCORING SYSTEM (choose one):
  Standard (3/1/0)      -- Win = 3 pts, Draw = 1 pt, Loss = 0 pts
  Win/Loss (1/1/0)      -- Win = 1 pt, Draw = 1 pt, Loss = 0 pts
  Custom Points         -- You set the exact points for Win, Draw, and Loss

GROUP SETTINGS (choose one):
  Teams Per Group       -- e.g. 4 means each group holds 4 teams
  Balance Groups Evenly -- spread teams equally across a fixed number of groups

Click Create Tournament. The new tournament opens automatically.


================================================================================
  5. MANAGING TEAMS
================================================================================

Inside a tournament, go to the Teams & Groups tab.

ADDING TEAMS
  Type the team name in the input at the bottom and press Add Team (or Enter).
  Optionally give the team a jersey number (#) -- shown in standings.

RENAMING A TEAM
  Click the pencil (edit) button next to any team chip.
  Type the new name and confirm.
  The rename cascades everywhere automatically: group lists, all match
  records, knockout brackets, standings -- the old name is replaced throughout.

REMOVING A TEAM
  Click the x button on the team chip.

ASSIGNING PLAYERS TO A TEAM
  Click the team name itself to open the Player Assignment Modal (PAM).
  Players with names similar to the team name are suggested automatically.
  Click any player row to instantly add or remove them -- no Save button needed.
  Click Done when finished.


================================================================================
  6. GROUPS & FIXTURES
================================================================================

AUTO-ASSIGNING GROUPS
  Once you have teams registered, click "Auto-Assign Teams to Groups".
  Teams are distributed according to your group settings (fixed size or balanced).

DRAG & DROP
  You can drag teams between groups to adjust assignments manually.
  Click a group label to rename it.

GENERATING FIXTURES
  Click "Generate Group Fixtures".
  This creates a complete round-robin schedule within each group -- every team
  plays every other team exactly once. Fixtures are interleaved across groups
  so the same team never appears in back-to-back matches.

  Note: if any match already has a score recorded, fixtures will NOT be
  regenerated -- this protects your existing results. Delete scored matches
  manually first if you need to start over.

ADDING MATCHES MANUALLY
  On the Fixtures tab, use the "Add Group Match" form at the top to add a
  specific match to any group.


================================================================================
  7. SCORING MATCHES
================================================================================

On the Fixtures tab, each group has a standings table and a match list.

ENTERING A SCORE
  Each match row has two score input boxes and three buttons:
    Save  -- saves scores, keeps status as "scheduled"
    Live  -- marks the match as currently in progress (red pulsing border)
    FT    -- marks the match as completed and updates the standings table

  Standings update instantly as you mark matches FT.

STANDINGS COLUMNS
  # / Team / W / D / L / GD / Pts
  Sorted by points first, then goal difference.
  Teams that will advance to the knockout bracket are highlighted with a star.


================================================================================
  8. AUTO-SCHEDULING
================================================================================

After generating fixtures, click the "Auto-Schedule" button to assign actual
dates and times.

SETTINGS
  Start Date        -- date of the first match
  First Kick-off    -- time the first match starts
  Match Duration    -- how long each match lasts (minutes)
  Break Between     -- gap between matches (minutes)
  Day Starts At     -- earliest time matches can be scheduled
  Day Ends At       -- latest time a match can start
  Venues            -- comma-separated list e.g. "Pitch A, Pitch B, Main Ground"

HOW IT WORKS
  - Matches are scheduled in rounds. All round-1 fixtures happen before
    round-2, etc.
  - If you provide multiple venues, matches within the same round are spread
    across courts simultaneously.
  - When the day-end boundary is reached, scheduling continues the next day.
  - Only group-phase matches are scheduled -- knockout matches you schedule
    manually.


================================================================================
  9. KNOCKOUT MODE
================================================================================

Once group matches are underway or complete, open the Knockout tab.

LAUNCHING THE KNOCKOUT
  1. Choose "Teams advancing per group" -- e.g. Top 2 means the top 2 teams
     from every group advance. You can advance anywhere from Top 1 (winner
     only) up to the entire group.
  2. The standings preview shows who would qualify (marked with a star symbol).
     Non-qualifying teams are dimmed.
  3. Click "Launch Knockout".

This generates a full bracket padded to the next power of 2. If the number
of qualified teams is not a power of 2 (e.g. 6 teams -> 8 slots), BYE matches
are added and auto-completed immediately.

ROUND NAMES
  Automatically assigned: Round of 32, Round of 16, Quarter-Final,
  Semi-Final, Final.

ADVANCING WINNERS
  When you mark a knockout match as FT, the winner automatically advances
  into the next round (the next TBD slot fills in).
  Use "Advance Winners" to manually propagate all completed results if needed.

RE-LAUNCHING
  You can re-launch the knockout at any time with different settings.
  All existing knockout matches will be replaced.

ADDING KO MATCHES MANUALLY
  Use the "Add Knockout Match Manually" form at the bottom of the Knockout tab
  to add a specific match outside of the generated bracket.

PUBLIC VIEW
  The Knockout tab appears on the public site once the bracket is active,
  showing the full bracket with scores, TBD slots, and completed results.


================================================================================
  10. LEAGUES
================================================================================

A league groups multiple tournaments together. Players' points accumulate
across all tournaments in the league for a combined standings table.

CREATING A LEAGUE
  Click "Leagues" in the sidebar, fill in the name and optionally a sport
  and description, then click Create League.

ADDING TOURNAMENTS TO A LEAGUE
  Either select the league when creating a tournament, or edit an existing
  league and check the tournaments you want to include.

PUBLIC LEADERBOARD
  Click "Leaderboard" in the public site header.
  Select a league to see the player standings table with total points, wins,
  draws, losses, goals for/against, and goal difference.
  Click any row to expand it and see a breakdown by tournament.


================================================================================
  11. PLAYERS & LEADERBOARDS
================================================================================

Players are registered individuals who can be assigned to teams. Their points
accumulate across tournaments for the league leaderboard.

REGISTERING A PLAYER
  Click "Players" in the sidebar.
  Enter a name and optional bio/notes, then click Register Player.

ASSIGNING PLAYERS TO TEAMS
  In a tournament, click any team name to open the Player Assignment Modal.
  The system suggests players whose names match the team name (fuzzy matching).
  Click a player to add/remove them. Changes save instantly.

LEADERBOARD CALCULATION
  A player's points come from their team's standings in each tournament.
  If a player is on multiple teams (across different tournaments in the same
  league), all results are summed.


================================================================================
  12. MEDIA (PHOTOS & VIDEOS)
================================================================================

Each tournament has a Media tab in the admin panel.

UPLOADING
  Click "Click to browse" or drag and drop files onto the upload zone.
  Multiple files can be selected at once.
  Supported formats: JPEG, PNG, GIF, WebP, MP4, MOV, WebM
  Maximum file size: 200MB per file

MANAGING
  Thumbnails appear in a grid. Hover over a thumbnail to reveal the delete
  button. Click any thumbnail to open a full-screen preview. Videos play with
  full controls.

PUBLIC GALLERY
  The Gallery tab appears automatically on the public site once a tournament
  has any media. Spectators see a thumbnail grid and can click to open a
  lightbox. Left/right arrow buttons (and keyboard arrow keys) navigate
  between items. Press Escape to close.

STORAGE
  Files are stored in data/media/<tournament-id>/ on the server.
  They are served directly by the server -- no external storage needed.
  When a tournament is deleted, its media folder is deleted too.


================================================================================
  13. PUBLIC SITE — WHAT SPECTATORS SEE
================================================================================

The public site at http://yourserver:3000 requires no login.

TOURNAMENT LIST
  All tournaments are shown as cards with status (Live/Upcoming/Completed),
  team count, and match count. Live tournaments pulse red.
  Click any card to open that tournament.

INSIDE A TOURNAMENT
  Up to four tabs are available:

  Groups & Tables
    Full standings tables for every group. Click a team name to see its
    profile card (recent results, upcoming fixtures, stats, follow button).
    Teams advancing to the knockout bracket are highlighted.

  Fixtures
    Match cards organised in columns by group. Live matches have a red border.
    If you follow any teams in this tournament, a "My Teams" column appears
    first showing only your teams' matches.

  Knockout
    The full bracket displayed left-to-right, earliest round first.
    Shows scores, completed results (winner highlighted in gold), and TBD
    slots for upcoming rounds.
    A "Qualified Teams" grid above the bracket shows who advanced from groups.

  Gallery
    Photo and video grid (only appears if media has been uploaded).
    Click to open fullscreen lightbox. Navigate with arrow keys or buttons.

BACK BUTTON
  The browser back button works -- the URL updates as you navigate so you can
  share a direct link to any tournament.


================================================================================
  14. FAVOURITES & NOTIFICATIONS
================================================================================

FOLLOWING TEAMS (public site)
  Open any team's profile (click the team name anywhere), then click the star.
  Followed teams are marked throughout the site.

FOLLOWING TAB
  The Following tab appears in a tournament if you have followed any teams.
  Shows each followed team's position, stats, and their next 3 fixtures.
  Your followed teams' matches are also shown first in the Fixtures tab,
  highlighted with a gold border.

COOKIE CONSENT
  On first visit, a banner asks for consent to store a small identifier on
  your device. This is just a random code (not personal data) used to remember
  which teams you follow. Click Accept to enable following, or No Thanks to
  dismiss (following won't be available without it).

NOTIFICATIONS
  If you have followed teams and live matches are scheduled, an
  "Enable Notifications" button appears in the header.
  Click it to grant permission -- you will receive a notification 5 minutes
  before any of your followed teams' scheduled matches.

  Works in modern browsers (Chrome, Firefox, Safari) and in the Android
  app wrapper if installed.

PERSISTENCE
  Followed teams are saved to the server and sync across devices -- if you
  follow a team on your phone, it appears on your laptop too.
  Favourites expire and are automatically cleaned up after 14 days of
  inactivity.


================================================================================
  15. HTTPS SETUP
================================================================================

The server auto-detects SSL certificates on startup. Place them in the ssl/
folder and restart -- no code changes needed.

OPTION A -- Let's Encrypt (free, for public domains)
  sudo apt install certbot
  sudo certbot certonly --standalone -d yourdomain.com

  Then copy the certificates:
  sudo cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem ssl/cert.pem
  sudo cp /etc/letsencrypt/live/yourdomain.com/privkey.pem  ssl/key.pem
  sudo chown $(whoami) ssl/*.pem

  Restart the server. HTTPS starts on port 3443, HTTP redirects automatically.

OPTION B -- Self-signed (for local network only)
  openssl req -x509 -newkey rsa:4096 -keyout ssl/key.pem -out ssl/cert.pem \
    -days 365 -nodes -subj "/CN=localhost"

  Browsers will show a warning -- click Advanced -> Proceed to accept it once.

AUTO-RENEWAL (Let's Encrypt)
  Certificates expire after 90 days. Add to root crontab (sudo crontab -e):

  0 3 1 * * certbot renew --standalone --pre-hook "pkill node" \
    --post-hook "cp /etc/letsencrypt/live/yourdomain.com/fullchain.pem \
    /path/to/hyderi/ssl/cert.pem && cp \
    /etc/letsencrypt/live/yourdomain.com/privkey.pem \
    /path/to/hyderi/ssl/key.pem && node /path/to/hyderi/server.js &"


================================================================================
  16. GITHUB SYNC
================================================================================

The server automatically fetches key files from a GitHub repository on startup
and every 6 hours. This protects files like the licence from being accidentally
deleted -- they are restored from GitHub next time the server starts.

REPOSITORY
  https://github.com/Blaze12345-deluxe/hyderi-sports-licence

FILES SYNCED
  From the repo root:
    licence.md  ->  saved to  files/licence.md  (public licence page)
    README.txt  ->  saved to  files/README.txt  (this file)

If GitHub is unreachable (no internet, repo is private, etc.), the server
starts normally and uses whatever files exist locally. The sync is always
silent -- it never blocks startup or causes errors.

OVERRIDING THE REPO URL
  GITHUB_RAW=https://raw.githubusercontent.com/YourUser/YourRepo/main node server.js


================================================================================
  17. CONFIGURATION REFERENCE
================================================================================

All configuration is done via environment variables. Defaults work out of the
box for local use.

  Variable           Default                         Description
  -----------------------------------------------------------------------
  PORT               3000                            HTTP port
  HTTPS_PORT         3443                            HTTPS port
  ADMIN_PASSWORD     HyderiAdmin321                  Admin login password
  SCRYPT_PEPPER      hyderi-sports-pepper-v1         Auth pepper (do not
                                                     change after first run)
  GITHUB_RAW         hyderi-sports-licence repo      Base URL for sync
  ALLOWED_ORIGINS    (empty)                         Extra CORS origins

EXAMPLES
  # Change password
  ADMIN_PASSWORD=mysecret node server.js

  # Run on standard ports (requires root or iptables redirect)
  PORT=80 HTTPS_PORT=443 node server.js

  # Override GitHub sync repo
  GITHUB_RAW=https://raw.githubusercontent.com/me/myrepo/main node server.js


================================================================================
  18. FILE STRUCTURE
================================================================================

  server.js               The entire backend -- one file, zero npm packages
  admin/
    index.html            Admin panel (all CSS and JS inline)
  public/
    index.html            Public site (all CSS and JS inline)
    manifest.json         PWA manifest for add-to-home-screen
  files/
    licence.md            Licence text (shown on the public /licence page)
    README.txt            This file (synced from GitHub)
    logo.png              Site logo
  ssl/
    cert.pem              SSL certificate (place here for HTTPS)
    key.pem               SSL private key (place here for HTTPS)
  data/
    tournaments.json      All tournament data
    leagues.json          All league data
    players.json          All player data
    tournament_<id>.json  Per-tournament backup file
    users/
      <uid>.json          Per-user favourites (auto-deleted after 14 days)
    media/
      <tournament-id>/    Uploaded photos and videos per tournament


================================================================================
  19. TROUBLESHOOTING
================================================================================

CANNOT CONNECT TO ADMIN
  - Make sure the server is running (check your tmux session)
  - Try http://localhost:3000/admin (not https if no certs are set up)
  - Check the terminal for error messages

WRONG PASSWORD
  The server prints at startup whether it is using the default password or one
  from an environment variable. The default is: HyderiAdmin321

SCORES NOT SAVING
  Make sure you click FT (or Save/Live) -- scores are not auto-saved as you
  type them into the boxes.

FIXTURES NOT REGENERATING AFTER DRAGGING GROUPS
  This is intentional: if any match already has a score, fixtures are
  protected. Delete scored matches first if you need to regenerate.

MEDIA UPLOAD FAILS
  - File must be under 200MB
  - Only JPEG, PNG, GIF, WebP, MP4, MOV, WebM are accepted
  - Check the browser console for the specific error message

GITHUB SYNC NOT WORKING
  - The repo must be public
  - Files must exist at the root of the main branch: licence.md and README.txt
  - No internet connection = sync silently skipped, local files used

FAVOURITES NOT PERSISTING
  The user must have accepted the cookie consent banner.
  After accepting, their ID is stored in a cookie and favourites sync to the
  server. Without the cookie, nothing is saved.

DATA RECOVERY
  If tournaments.json is corrupted, individual tournament_<id>.json files in
  data/ can be used to recover. Each contains the full data for one tournament.

RASPBERRY PI -- SERVER CRASHED
  Attach to the tmux session:   tmux attach -t hyderi
  Restart if needed:            node server.js

================================================================================
  Built by Rezahusein Karim
================================================================================
