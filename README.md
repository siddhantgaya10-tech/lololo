# GUARDIAN AI – Stage 3 (AI decision engine)

Smart Protection. Healthier Digital Habits.

## Build the APK for free (no Android Studio needed)
1. Create a GitHub account and a new **private** repository.
2. Upload the contents of this folder (keep the `.github` folder; on the web use "Add file → Upload files" and drag the whole folder contents).
3. Open the **Actions** tab → **Build debug APK** → **Run workflow**.
4. When it finishes, open the run and download **guardian-ai-debug-apk** from *Artifacts*. Unzip it and install `app-debug.apk` (allow "install unknown apps").

## What Stage 1 contains
- Parent account (name, email, 6-digit PIN) stored locally; PIN and recovery code are salted PBKDF2-SHA256 hashes (210k iterations). Nothing is stored in plain text.
- PIN is separate from the phone lock; the phone's lock-screen credentials never open the parent area.
- Lockout after 5 wrong tries (30 s, doubling to 15 min), shared by PIN and recovery-code guesses.
- One-time recovery code (16 chars) to reset a forgotten PIN; regenerating it needs the current PIN.
- Parent session closes on leaving the app and after 10 min idle; every setting change checks it.
- FLAG_SECURE blocks screenshots of PIN/recovery screens; backups of app data are disabled.
- Room database: account, app rules, schedules, mode, approved channels, security events.
- Parent area: learning mode, per-app daily limits (from launchable apps), bedtime/study schedules, PIN change, security log.
- Unit tests for hashing, PIN policy and recovery codes.

## Not yet included (later stages)
Usage monitoring and blocking, AI classifier, Device Admin/Owner protection, notifications, dashboards with charts.
Settings are saved but not enforced until Stage 2.


## Stage 2 additions
- Foreground-app monitoring with UsageStatsManager (package name + duration only), run by a foreground service
  (1 s polling while the screen is on, 5 s while off), restarted after reboot/update and by a 15-min WorkManager watchdog.
- Full-screen restriction screen (limit reached / bedtime / study time) with: go home, emergency call (opens the dialer, which is never blocked),
  and parent access (PIN) to grant a 15/30/60 minute extension.
- Policy engine (unit-tested): extension > bedtime > study (entertainment apps) > daily app limit.
- Daily usage saved locally (used on the Apps tab; weekly reports come with the dashboards stage).
- Break reminders after continuous phone use (Setup tab).
- New **Setup** tab: permission status and grant buttons.
- Logs AUTO_TIME_DISABLED if automatic date & time is switched off (usage days follow the phone clock).

## After installing
1. Open the app, create the parent account, save the recovery code.
2. **Setup** tab: grant Usage access and Display over other apps (required), notifications and battery (recommended).
3. **Apps** tab: tap an app and pick a daily limit. **Schedules** tab: add bedtime / study time.

## Known limits (honest list)
- Enforcement works by detecting the foreground app and covering it; a child who finds Settings can still revoke permissions or uninstall until Stage 4 (Device Admin/Owner + alerts).
- Study mode / study time block apps that have an entertainment-category rule (games, video, social, audio). Letting educational YouTube through needs the Stage 3 classifier.
- Limits are per app (category group limits like "all games 60 min" are not in yet).
- Android 12+ may refuse background service restarts on some phones; the watchdog still enforces once per 15 min in that case.


## Stage 3 additions
- **Classifier** (on-device, rule-based, transparent): returns ALLOW / RESTRICT / UNCERTAIN for a YouTube activity.
  Signals: video title, approved channels, mode (Experiment mode also counts hands-on words like arduino/circuit/robot),
  short-form detection, and parent decisions. Keywords alone never allow a video: ALLOW needs a clear educational signal
  with no entertainment wording *and* a reliably read title, or an approved channel, or a parent override.
- **Decision engine**: ALLOW (and not over the daily educational allowance) is excluded from entertainment limits and study blocks.
  RESTRICT and UNCERTAIN follow your normal rules. The AI can never lift bedtime, schedules or a parent decision.
- **Review tab**: videos the AI restricted or couldn't judge (after 20 s on screen), with the reasons. Mark Educational / Entertainment,
  approve a channel, or reset. Decisions are remembered per video. Allowed videos are never stored.
- **Educational apps**: in the Apps tab choose "Educational app (always allowed)". Their time counts as learning time, not entertainment.
- **Block screen**: after parent PIN, "This video is educational" appears when the blocked video is known.
- **Activity detection** (optional Accessibility service) with an in-app disclosure; only events from the YouTube apps; text read only from the
  title/channel views. Educational time is stored per day for the upcoming dashboards.
- Setup tab: daily educational allowance (No cap / 90m / 2h / 3h).

## Honest limits of Stage 3
- YouTube's internal view ids are not a public API. If YouTube changes them, titles stop being found and videos fall back to UNCERTAIN (restricted per rules), never to a wrong ALLOW.
- Fullscreen playback hides the title, so a video context is kept for up to 30 min since it was last seen, or until the child leaves YouTube or the screen turns off.
- Without the Accessibility service, YouTube is treated as entertainment (use "Educational app" or an approved-channel workflow with the service on).
- Other browsers/websites are not classified; Chrome-based learning sites are not covered.
