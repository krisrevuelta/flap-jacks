# Installing Claude Code for use with Cursor

This document records how Claude Code was installed on this machine and how to use it alongside the Cursor IDE.

**Environment (at install time):**

- macOS (darwin 25.3.0)
- Node.js v24.16.0 (`/usr/local/bin/node`)
- npm 11.13.0 (`/usr/local/bin/npm`)
- Cursor IDE (integrated terminal)

---

## Tools installed

| Tool | Version / location | Purpose |
|------|-------------------|---------|
| **Node.js** | v24.16.0 | Runtime for the Claude Code CLI |
| **npm** | 11.13.0 | Package manager used to install Claude Code |
| **@anthropic-ai/claude-code** | 2.1.150 | Anthropic’s Claude Code CLI (`claude` command) |

**Install location (user prefix, not system-wide):**

- Binary: `~/.npm-global/bin/claude`
- Package: `~/.npm-global/lib/node_modules/@anthropic-ai/claude-code`

---

## Steps completed

### 1. Attempt global install (failed)

```bash
npm install -g @anthropic-ai/claude-code
```

This targeted `/usr/local/lib/node_modules`, which is owned by `root` on this machine. The install failed with `EPERM: operation not permitted`.

### 2. Install to a user-owned prefix (succeeded)

```bash
mkdir -p "$HOME/.npm-global"
npm install -g @anthropic-ai/claude-code --prefix "$HOME/.npm-global"
```

### 3. Verify installation

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
claude --version
# → 2.1.150 (Claude Code)

npm list -g --prefix "$HOME/.npm-global" @anthropic-ai/claude-code
# → @anthropic-ai/claude-code@2.1.150
```

### 4. Verify authentication (pending)

```bash
claude auth status
# → "loggedIn": false (login not completed in this setup session)

claude -p "Reply with exactly: OK"
# → Not logged in · Please run /login
```

### 5. Use with Cursor

1. Open **Terminal → New Terminal** in Cursor (or `` Ctrl+` ``).
2. Ensure `claude` is on your `PATH` (see [PATH setup](#path-setup) below).
3. Start Claude Code:

   ```bash
   claude
   ```

4. Sign in (required once):

   - In the interactive session: `/login`
   - Or from the shell:

     ```bash
     claude auth login              # Claude subscription (Pro/Max/Team)
     claude auth login --console    # Anthropic Console / API billing
     claude auth login --sso        # SSO
     ```

5. Confirm login:

   ```bash
   claude auth status
   claude -p "Reply with exactly: OK"
   ```

---

## Issues encountered and fixes

### Issue: `EPERM` on `npm install -g`

**Symptom:**

```
npm error code EPERM
npm error path /usr/local/lib/node_modules/@anthropic-ai
```

**Cause:** `/usr/local/lib/node_modules` is owned by `root`; a normal user cannot write there.

**Fix (used):** Install to a user prefix:

```bash
npm install -g @anthropic-ai/claude-code --prefix "$HOME/.npm-global"
```

**Alternative:** System-wide install with elevated permissions (only if you prefer `/usr/local`):

```bash
sudo npm install -g @anthropic-ai/claude-code
```

---

### Issue: `command not found: claude` in a new terminal

**Cause:** `~/.npm-global/bin` was not added to the shell `PATH` in `~/.zshrc` or `~/.zprofile`.

**Fix:** Add to `~/.zshrc`:

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

Then reload:

```bash
source ~/.zshrc
```

Or set `PATH` for the current session only:

```bash
export PATH="$HOME/.npm-global/bin:$PATH"
```

---

### Issue: Cannot run interactive `claude` from Cursor Agent chat

**Cause:** Claude Code’s default mode is an interactive TUI; the Agent’s non-interactive shell cannot host that session.

**Fix:** Run `claude` in Cursor’s **integrated terminal**, not via the Agent run command for day-to-day use.

---

### Issue: API calls fail with “Not logged in”

**Symptom:**

```
Not logged in · Please run /login
```

**Fix:** Complete authentication in your terminal (`claude auth login` or `/login` inside `claude`). Re-check with `claude auth status`.

---

### Issue: `claude doctor` appears to hang

**Cause:** `claude doctor` is interactive and may wait on TTY / workspace checks when run without a real terminal.

**Fix:** Use non-interactive checks instead:

```bash
claude --version
claude auth status
claude -p "test"    # after login
```

---

## PATH setup

Add once to `~/.zshrc`:

```bash
# Claude Code (npm user prefix)
export PATH="$HOME/.npm-global/bin:$PATH"
```

---

## Quick reference

| Task | Command |
|------|---------|
| Version | `claude --version` |
| Auth status | `claude auth status` |
| Log in | `claude auth login` or `claude` then `/login` |
| Log out | `claude auth logout` |
| One-shot prompt | `claude -p "your prompt"` |
| Help | `claude --help` |

---

## Status checklist

- [x] Claude Code package installed (`2.1.150`)
- [x] `claude` binary available under `~/.npm-global/bin`
- [ ] `PATH` persisted in `~/.zshrc` (recommended)
- [ ] Authenticated (`claude auth status` → `"loggedIn": true`)
- [ ] Smoke test passed (`claude -p "Reply with exactly: OK"`)

---

## Optional: system-wide install

If you later want the binary in `/usr/local/bin` without a custom prefix:

```bash
sudo npm install -g @anthropic-ai/claude-code
```

You may then remove the user-prefix copy:

```bash
npm uninstall -g @anthropic-ai/claude-code --prefix "$HOME/.npm-global"
```

---

*Generated from the Cursor Agent setup session on 2026-05-23.*
