# Installing gog + OAuth setup

`gog` is a Go binary. Install the binary first, then do the one-time OAuth dance.

## Verify

After install:

```bash
gog --version
```

## Installing the binary

### Homebrew (macOS and Linux)

```bash
brew install openclaw/tap/gogcli
```

The formula installs the `gog` binary on `$PATH`.

### Manual binary download

Releases are published on the upstream project; download the right binary for your OS/arch and put it on `$PATH`:

```bash
# Linux x86_64 — pin to the current release tag from https://github.com/openclaw/gogcli/releases
curl -L -o /tmp/gogcli.tar.gz \
  https://github.com/openclaw/gogcli/releases/download/v0.21.0/gogcli_0.21.0_linux_amd64.tar.gz
tar -xzf /tmp/gogcli.tar.gz -C /tmp
sudo install /tmp/gog /usr/local/bin/gog
```

### Docker / GHCR

```bash
docker run --rm ghcr.io/openclaw/gogcli:<pinned-tag> --version
```

Source builds live at github.com/openclaw/gogcli for anyone who wants them.

## One-time OAuth client setup

`gog` is a Google API client; like every Google API client, it needs **your own** OAuth credentials. Google does not allow distributed binaries to ship their own client secret in this category.

### Step 1 — Create an OAuth client in Google Cloud Console

1. Open https://console.cloud.google.com.
2. **Create a project** (or pick an existing one).
3. **APIs & Services → Enabled APIs & services** → Enable the APIs you want to use:
   - Gmail API
   - Google Calendar API
   - Google Drive API
   - People API (for Contacts)
   - Google Sheets API
   - Google Docs API
4. **APIs & Services → OAuth consent screen** → configure as **External**, add yourself as a test user.
5. **APIs & Services → Credentials** → **Create Credentials** → **OAuth client ID** → **Application type: Desktop app** → name it (e.g. "gog CLI") → Create.
6. **Download JSON** for the new client. You'll get a file named like `client_secret_<long-id>.apps.googleusercontent.com.json` — save it somewhere safe (e.g. `~/.config/gogcli/client_secret.json`).

### Step 2 — Hand the credentials to gog

```bash
gog auth credentials ~/.config/gogcli/client_secret.json
```

### Step 3 — Add each Google account

For each Google account you want `gog` to access:

```bash
gog auth add you@gmail.com --services gmail,calendar,drive,contacts,sheets,docs
```

A browser window opens. Sign in as `you@gmail.com`, grant the requested scopes, and the consent page returns a code that `gog` captures automatically.

Repeat for additional accounts:

```bash
gog auth add you-work@gmail.com --services gmail,calendar,sheets,docs
```

### Step 4 — Confirm

```bash
gog auth list
```

…lists the configured accounts and which services each has scoped.

```bash
gog gmail search 'newer_than:7d' --max 5
```

…should return your last 5 emails from the past week.

## Optional: keyring env file pattern

Some setups store the path to the OAuth keyring and (optionally) an encryption key in a sourced env file, e.g. `~/.config/gogcli/keyring.env`:

```bash
export GOG_KEYRING_FILE=~/.config/gogcli/keyring.json
export GOG_KEYRING_PASSPHRASE_CMD="pass show gog/keyring"
```

The file is NOT auto-sourced. To use it, either:

- `source ~/.config/gogcli/keyring.env` in every new shell, or
- Add the source line to `~/.zshrc` / `~/.bashrc`.

Without this pattern, `gog` uses its default keyring location and you don't need to source anything.

## Troubleshooting

- **`not authenticated` / `no credentials found`** — you forgot to `source` the keyring env file, OR you skipped `gog auth credentials`, OR the OAuth client JSON wasn't where `gog` expects.
- **`access blocked` on the OAuth browser flow** — your Google account isn't listed as a test user on the OAuth consent screen. Add it under **APIs & Services → OAuth consent screen → Test users**.
- **`API not enabled`** — go back to **APIs & Services → Enabled APIs & services** and enable the API for the service you're calling.
- **`quota exceeded`** — Google's per-project per-day quotas are surprisingly low for some APIs. Increase quota in Cloud Console or wait until the daily reset.
