# FitLog

Offline-first fitness logging app (workouts, weight & measurements, meals).
All data is stored in the browser's localStorage on the device. Nothing is sent anywhere.

## Publish to GitHub Pages (copy-paste)

```bash
cd fitlog
git init -b main
git add .
git commit -m "FitLog initial version"
gh repo create fitlog --public --source=. --push
# or, without the gh CLI:
#   create an empty repo named "fitlog" on github.com, then:
#   git remote add origin https://github.com/<your-username>/fitlog.git
#   git push -u origin main
```

Then: GitHub repo -> Settings -> Pages -> Source: "Deploy from a branch" -> Branch: `main`, folder `/ (root)` -> Save.

The app appears at `https://<your-username>.github.io/fitlog/` after about a minute.

## Install on iPhone

1. Open that URL in **Safari** (not Chrome).
2. Share -> **Add to Home Screen** -> Add.
3. Launch from the home-screen icon.

## Promoting a change

```bash
git add . && git commit -m "change" && git push
```
GitHub Pages redeploys. On the phone, close the app fully (swipe up from the app switcher)
and reopen it twice: the service worker fetches the new version on the first launch and
activates it on the second.

## Notes

- The repo holds **code only**. Your logged data never leaves the phone, so a public repo
  is fine. A private repo needs a paid GitHub plan for Pages.
- Back up from the app: gear icon -> "Back up". Clearing Safari website data erases the log.

---

## Optional: a local LLM for meal parsing

The app ships with a 6,200-food database plus ~100 curated dishes, which covers most
entries offline with no AI at all. For free-text it doesn't recognise, you can point the
app at **any local LLM that speaks the OpenAI API** — Ollama, LM Studio, or a llama.cpp
server. Your meal text goes only to that address. Nothing reaches Anthropic or any vendor.

### On the Mac (Ollama)

```bash
brew install ollama
ollama pull qwen2.5:7b          # or llama3.1:8b — 7-8B is plenty for this job
```

The app is served over HTTPS from GitHub Pages, and browsers refuse to call a plain
`http://` address from an HTTPS page. Tailscale gives the Mac a real HTTPS name and also
lets the phone reach it from outside the house:

```bash
brew install --cask tailscale
tailscale up
tailscale serve --bg --https=443 http://127.0.0.1:11434
tailscale serve status          # prints the https://<machine>.<tailnet>.ts.net URL
```

The `tailscale serve` flags have changed between versions — run `tailscale serve --help`
if that line is rejected.

Ollama also has to accept requests from the app's origin:

```bash
OLLAMA_ORIGINS="https://<your-username>.github.io" ollama serve
```

Install the Tailscale app on the iPhone and sign in with the same account.

### In the app

Gear icon -> "Local AI meal parser":

- Endpoint: `https://<machine>.<tailnet>.ts.net/v1`
- Model: `qwen2.5:7b`
- Key: leave blank

Press **Test connection**, then **Save changes**.

### When the Mac is off

Nothing breaks. The app falls back to the database, tells you which engine it used, and
any meal saved without numbers is marked "needs numbers". Tap **Fill in unestimated
meals** later and it reruns them through whichever parser is available.
