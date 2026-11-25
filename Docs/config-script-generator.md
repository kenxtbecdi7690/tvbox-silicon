# Yingxia Config Script Generator (AI-Ready Version)

> Usage: Copy the prompt for your target device, paste it to any AI, fill in the `<...>` parts, and generate directly.
> Version pinned: **Yingxia 1.8.15** (the current version on the official website; when the version changes, re-check it using Section 6 at the end).
> Official rule sources: Developer Guide `Docs/DEVELOPMENT.md` (main branch, read 2026-11-24), README, official website https://yingxia.getmegaportal.com
> Repository: https://github.com/Canghangbuxiaoyun/tvbox-silicon (GPL-3.0). The repository already ships an AI skill `.agents/skills/install-yingxia` (for installing the app). This file is a separate one, for writing config scripts.

---

## 1. Official Rules (Common to All Four Devices — Must Be Followed When Generating with AI)

1. **Subscription format**: TVBox JSON, imported by URL; multiple subscriptions are merged into one source list; keep plugin fields such as `ext`, `spider`, and rules; relative addresses are resolved against the subscription's own address; the app ships with no default sources; if a refresh fails, keep the last working configuration.
2. **Source types**: type 1 / type 4 are JSON API sources (home, categories, search, pagination, detail, playback, episode selection, direct links); type 3 is a plugin source: a `.js` api is a JavaScript crawler, a `.py` api (or the legacy `py_` + ext as a script) is a Python crawler, and any other api is a JAR class.
3. **Runtime order**: Only list sources that the device can run, in the order JSON API → Python → JavaScript → JAR; multiple sources for the same site are merged into one channel; during search, each runtime runs in its own lane, so a slow JVM does not block the others.
4. **Never write (officially unsupported)**: Full CatVod / Android API compatibility, arbitrary desktop JVM applications, web video sniffing, magnet links, multi-quality playback responses, XML collection APIs, EPG, cloud sync.
5. **Request timeouts**: Playback 90 seconds, search 60 seconds, other requests 150 seconds; on Mac, a single JS / Python subprocess request is 60 seconds; on timeout, report `source_timeout`.
6. **CENC-encrypted MP4**: Scripts can use `CENCStreams` to return an encrypted MP4 + content key; Range requests must be supported; the forward window is about 60 seconds; decrypted segments have a 16 MB cap; AES-CTR decryption.
7. **Sign-in**: When a source entry carries `yingxia.signIn`, the script provides an `account` function that produces a QR code, and the app displays it and saves the signed-in state.
8. **Plugin downloads**: Shared cache, SHA-256 verification; when a JAR changes content at the same URL, open a new session.
9. **Contribution rules**: Docs, comments, and developer diagnostics are written in English; UI copy goes through localization resources; GPL-3.0; the player itself provides no content, and generated sources should only cover sites the user has the right to access.

---

## 2. Runtime Comparison by Device

| Device | OS | JSON API | Python | JavaScript | JAR | Notes |
|---|---|---|---|---|---|---|
| Mac | macOS 15+ | ✓ | ✓ (built-in CPython; CatVod `base.spider` + requests / lxml / bs4 / pyquery / pycryptodome / cryptography; `localProxy` via a loopback port) | ✓ (built-in Node) | ✓ (built-in desktop JVM) | One reusable subprocess per source, up to 3, stopped after 5 minutes idle; JARs share one long-lived JVM; all run in the crawler sandbox |
| iPhone | iOS 17+ | ✓ | ✗ No Python | ✓ (QuickJS; HTTP, HTML parsing, module loading, preference adapters) | ✓ (built-in interpreted JVM, in-process) | Sources are not guaranteed to be identical to Mac |
| iPad | iOS 17+ | ✓ | ✗ No Python | ✓ (QuickJS) | ✓ (built-in JVM) | Same runtime as iPhone |
| Apple TV | tvOS 17+ | ✓ | ✗ No Python | ✓ (QuickJS) | ✓ (built-in JVM) | Search uses native `.searchable`; search starts only after input pauses for 650ms; Siri Remote focus navigation; diagnostic logs in `Library/Caches/com.tvbox.yingxia/Logs` |

- iPhone, iPad, and Apple TV share the same plugin runtime; Mac has the most complete runtime.
- Built-in lightweight ports (`Sources/TVCore/ScriptAssets/sites/`) run QuickJS first, with the original plugin as a fallback.

---

## 3. Generator Prompts (One per Device — Copy Directly to an AI)

### Mac (macOS 15+, Yingxia 1.8.15)

```text
You are the Yingxia config script generator. Target device: Mac (macOS 15+), Yingxia 1.8.15.
Official rule source: https://github.com/Canghangbuxiaoyun/tvbox-silicon/blob/main/Docs/DEVELOPMENT.md

Task: Generate for <...>.

You must follow these rules:
- Mac can run all four source types: JSON API, Python (built-in CPython; base.spider, requests, lxml, bs4, pyquery, pycryptodome, cryptography are available; localProxy goes via a loopback port), JavaScript (built-in Node), JAR (built-in desktop JVM).
- Subscriptions use TVBox JSON; keep the ext, spider, and rules fields; relative addresses are resolved against the subscription's own address.
- Route plugin sources by rule: .js is a JS crawler, .py is a Python crawler, any other api is a JAR class.
- Never write: full CatVod/Android API compatibility, web video sniffing, magnet links, multi-quality playback responses, XML collection APIs, EPG, cloud sync.
- Timeouts: playback 90 seconds, search 60 seconds, other requests 150 seconds; a single JS/Python subprocess request is 60 seconds. Do not retry in an infinite loop on timeout.
- For sources that need sign-in: add yingxia.signIn to the entry, and provide an account function that produces a QR code.
- For encrypted MP4, use CENCStreams to return the encrypted address + content key, and Range requests must be supported.

Output:
1. The complete, usable subscription JSON or full script text — no ellipses.
2. A self-check list: for each official rule, whether it is met, and if not, why not.
3. Verification command: Scripts/test-sources.py [subscription URL], with the expected result.
```

### iPhone (iOS 17+, Yingxia 1.8.15)

```text
You are the Yingxia config script generator. Target device: iPhone (iOS 17+), Yingxia 1.8.15.
Official rule source: https://github.com/Canghangbuxiaoyun/tvbox-silicon/blob/main/Docs/DEVELOPMENT.md

Task: Generate for <...>.

You must follow these rules:
- iPhone has no Python: do not generate Python crawlers; when a crawler is needed, write a JS crawler (QuickJS: HTTP, HTML parsing, module loading, preference adapters) or use a JAR.
- The runnable source order is JSON API → JavaScript → JAR; only list sources this device can run.
- All other rules are the same as the common rules (see Section 1 of this file): subscription format, source types, the never-write list, timeouts, CENC, sign-in.
- JARs run in the built-in interpreted JVM, and the first conversion is slow — do not put them on the first-screen critical path.

Output:
1. The complete, usable subscription JSON or full script text — no ellipses.
2. A self-check list: whether Python or Mac-only capabilities were used, and whether anything from the never-write list was written.
3. Verification command: swift run TVBoxProbe --site-check <source key>, with the expected result.
```

### iPad (iOS 17+, Yingxia 1.8.15)

```text
You are the Yingxia config script generator. Target device: iPad (iOS 17+), Yingxia 1.8.15.
Official rule source: https://github.com/Canghangbuxiaoyun/tvbox-silicon/blob/main/Docs/DEVELOPMENT.md

Task: Generate for <...>.

You must follow these rules:
- iPad uses the same runtime as iPhone: no Python; JS runs on QuickJS, JARs run on the built-in interpreted JVM.
- The runnable source order is JSON API → JavaScript → JAR; only list sources this device can run.
- All other rules are the same as the common rules (see Section 1 of this file): subscription format, source types, the never-write list, timeouts, CENC, sign-in.

Output:
1. The complete, usable subscription JSON or full script text — no ellipses.
2. A self-check list: whether Python or Mac-only capabilities were used, and whether anything from the never-write list was written.
3. Verification command: swift run TVBoxProbe --site-check <source key>, with the expected result.
```

### Apple TV (tvOS 17+, Yingxia 1.8.15)

```text
You are the Yingxia config script generator. Target device: Apple TV (tvOS 17+), Yingxia 1.8.15.
Official rule source: https://github.com/Canghangbuxiaoyun/tvbox-silicon/blob/main/Docs/DEVELOPMENT.md

Task: Generate for <...>.

You must follow these rules:
- Apple TV has no Python: do not generate Python crawlers; when a crawler is needed, write a JS crawler (QuickJS) or use a JAR.
- The runnable source order is JSON API → JavaScript → JAR; only list sources this device can run.
- Search goes through native .searchable: search only after input pauses for 650ms; the results page must keep the search state, and going back must not lose the results.
- All other rules are the same as the common rules (see Section 1 of this file): subscription format, source types, the never-write list, timeouts, CENC, sign-in.
- Diagnostic logs are in Library/Caches/com.tvbox.yingxia/Logs — leave error messages that can be located when writing scripts.

Output:
1. The complete, usable subscription JSON or full script text — no ellipses.
2. A self-check list: whether Python or Mac-only capabilities were used, and whether anything from the never-write list was written.
3. Verification command: swift run TVBoxProbe --site-check <source key>, with the expected result.
```

---

## 4. Free Conversion (Rules for Converting Between Devices — This Table Is What Makes Conversion Flexible)

| From → To | How |
|---|---|
| JSON API source → any device | No changes needed; works on all four devices. |
| Python crawler → iPhone / iPad / Apple TV | Must be rewritten as a JS crawler (QuickJS: HTTP, HTML parsing, module loading, preference adapters); or find a JAR for the same site. Python libraries (requests, bs4, etc.) cannot be carried over directly. |
| JS crawler → any device | Prefer using only capabilities QuickJS also has (HTTP, HTML parsing, module loading, preferences), and do not use Node-only APIs, so it works on all four devices; Node on Mac can do more, but once you use it, it only runs on Mac. |
| JAR → any device | Runs on all four devices; but on iOS / tvOS it is an interpreted built-in JVM, and the first conversion is slow (officially measured at about 9.6 seconds for the first conversion, about 3.9–4.7 seconds after caching) — do not put it on the first-screen critical path. |
| Mac-only → other devices | Check the comparison table first: anything using Python, Node-only APIs, or the desktop JVM must be converted using the three rows above. |

Conversion checklist (check each item after converting):
- [ ] Does the target device's runtime support this source type?
- [ ] Are the timeouts sufficient (playback 90 seconds, search 60 seconds, other requests 150 seconds)?
- [ ] Were any officially unsupported capabilities used?
- [ ] Was it run through the tools in Section 5?
- [ ] Was it verified on a real device? (A successful build does not mean it can play on a real device.)

---

## 5. Verification (Must Run After Generating)

```bash
# Full end-to-end check per source: home / categories / search / detail / playback + fetching the media URL
Scripts/test-sources.py [subscription URL]
# Results are written to build/site-test/results.json; passing only means the media URL responded, not that decoding and every HLS segment work; re-run for unstable sources

# Check a single source: directory, search, rankings, detail, time to first frame
swift run TVBoxProbe --site-check <source key> [show ID] [episode number]

# Audit a subscription: export a normalized subscription, then audit home and search
swift run TVBoxProbe --subscription-export <subscription URL> > build/subscription.json
python3 Scripts/audit-source-loading.py build/subscription.json --output build/source-audit

# JAR check without execution
swift run TVBoxProbe build/xxx.jar
python3 Scripts/inspect-plugin.py build/xxx.jar
```

---

## 6. What to Change When the Version Changes

1. **Version number**: Check the official website `yingxia.getmegaportal.com/releases.json` and the Get it table in the README, and update the version number in this document and in the four prompts.
- Note: TestFlight build numbers are `1.7.7 + release build number` (for example, 1.7.7 (31) corresponds to 1.7.12). Do not confuse them with release version numbers.
2. **Rules**: Re-read `Docs/DEVELOPMENT.md` (main branch), and check each item in Section 1 against it; if the runtime changes, update the comparison table first, then the four prompts.
3. **Currently pinned version**: 1.8.15 (the current version on the official website).
