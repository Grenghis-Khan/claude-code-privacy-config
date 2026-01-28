# Claude Code Privacy Mode Setup Guide

A comprehensive guide to configuring Claude Code with balanced privacy and functionality settings, similar to Cursor's Privacy Mode.

---

## Table of Contents

1. [Understanding the Privacy Landscape](#understanding-the-privacy-landscape)
2. [Step 1: Opt Out of Model Training](#step-1-opt-out-of-model-training)
3. [Step 2: Configure User-Level Privacy Settings](#step-2-configure-user-level-privacy-settings)
4. [Step 3: Configure Project-Level Privacy Settings](#step-3-configure-project-level-privacy-settings)
5. [Verification Checklist](#verification-checklist)
6. [What This Does (and Doesn't Do)](#what-this-does-and-doesnt-do)
7. [Troubleshooting](#troubleshooting)
---

## Understanding the Privacy Landscape

### What We're Achieving

This configuration gives you **balanced privacy and functionality** settings:

- ✅ Blocks access to sensitive files and directories (credentials, secrets, keys)
- ✅ Disables telemetry and error reporting
- ✅ Allows web access (WebFetch/WebSearch) for current documentation
- ✅ Allows dependency and build output reading for debugging
- ✅ Enables automatic updates for security patches
- ✅ Reduces data retention to 30 days (with account opt-out)
- ✅ Uses "ask" mode for critical operations (git push, package publishing)

### What Claude Code Can't Do

Unlike Cursor's Ghost Mode, Claude Code **cannot**:

- ❌ Process code entirely locally (always uses Anthropic's servers)
- ❌ Operate in a fully offline mode
- ❌ Bypass Anthropic's infrastructure completely

If you need true "air-gapped" development, Claude Code is not the right tool.

---

## Step 1: Opt Out of Model Training

This is **critical** - without this, your data retention is 5 years and can be used for training, instead of 30 days for trust & safety review only.

### For Personal Accounts (Free/Pro/Max)

1. Go to [https://claude.ai/settings/data-privacy-controls](https://claude.ai/settings/data-privacy-controls)
2. In **"Privacy settings"**, Toggle **OFF** the option: "Help improve Claude"

**Data retention after opt-out**: 30 days

### For Team/Enterprise Accounts

Good news: **Training is disabled by default** for commercial accounts.

1. Verify in your organization's [Console settings](https://console.anthropic.com/)
2. Navigate to **Organization Settings** → **Data Usage**
3. Confirm training is disabled

**Data retention**: Per your enterprise agreement (typically 30 days or zero retention)

### For API Users

If you're using Claude Code with API keys:

- Training is **automatically disabled**
- Data retention: 30 days for trust & safety only

---

## Step 2: Configure User-Level Privacy Settings

These settings apply to **all your projects** globally.

### File Location by Operating System

|Operating System|Path|
|---|---|
|**macOS**|`~/.claude/settings.json`|
|**Linux**|`~/.claude/settings.json`|
|**Windows**|`C:\Users\<YourUsername>\.claude\settings.json`|

### Creating the Settings File

**macOS / Linux:**

```bash
# Create the directory if it doesn't exist
mkdir -p ~/.claude

# Create/edit the settings file
nano ~/.claude/settings.json
```

**Windows (PowerShell):**

```powershell
# Create the directory if it doesn't exist
New-Item -ItemType Directory -Force -Path "$env:USERPROFILE\.claude"

# Create/edit the settings file
notepad "$env:USERPROFILE\.claude\settings.json"
```

### User Settings Configuration

> ⚠️ Security Note: This configuration uses `**/` glob patterns (e.g., `Read(**/.env)`) to block sensitive files **anywhere** in your project tree, including nested directories and packages. This ensures protection in monorepo structures like:
> - `packages/api/.env`
> - `apps/frontend/.env.local`
> - `services/auth/secrets/`
> 
>Always use `**/` patterns for security-critical files to ensure complete protection in monorepos and nested project structures.


Copy this configuration into your `~/.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      "WebFetch",
      "WebSearch",
      "Read(./node_modules/**)",
      "Read(./vendor/**)",
      "Read(./build/**)",
      "Read(./dist/**)",
      "Read(./.next/**)",
      "Read(./out/**)",
      "Read(./target/**)",
      "Read(**/package-lock.json)",
      "Read(**/yarn.lock)",
      "Read(**/pnpm-lock.yaml)",
      "Read(**/Gemfile.lock)",
      "Read(**/Pipfile.lock)",
      "Read(**/composer.lock)",
      "Read(**/Cargo.lock)",
      "Read(**/.env.example)",
      "Read(**/.env.template)",
      "Read(**/.env.sample)",
      "Read(**/*.example)",
      "Read(**/*.template)",
      "Read(**/*.sample)",
      "Edit(**/.env.example)",
      "Edit(**/.env.template)",
      "Edit(**/.env.sample)",
      "Edit(**/*.example)",
      "Edit(**/*.template)",
      "Edit(**/*.sample)",
      "Write(**/.env.example)",
      "Write(**/.env.template)",
      "Write(**/.env.sample)"
    ],
    "ask": [
      "Read(./.git/config)",
      "Edit(./.git/config)",
      "Write(./.git/config)",
      "Bash(git push*)",
      "Bash(rm -rf*)",
      "Bash(npm publish*)",
      "Bash(yarn publish*)",
      "Bash(pnpm publish*)",
      "Write(package.json)",
      "Write(Cargo.toml)",
      "Write(Gemfile)",
      "Write(requirements.txt)",
      "Write(composer.json)"
    ],
    "deny": [
      "Read(**/.env)",
      "Read(**/.env.local)",
      "Read(**/.env.development)",
      "Read(**/.env.production)",
      "Read(**/.env.staging)",
      "Read(**/.env.test)",
      "Read(**/secrets/**)",
      "Read(**/config/secrets/**)",
      "Read(**/config/credentials.json)",
      "Read(**/.aws/**)",
      "Read(**/.ssh/**)",
      "Read(**/.netrc)",
      "Read(**/id_ecdsa*)",
      "Read(**/id_ed25519*)",
      "Read(**/.gnupg/**)",
      "Read(**/.docker/config.json)",
      "Read(**/keystore*)",
      "Read(**/.kube/config)",
      "Read(**/ServiceAccount*.json)",
      "Read(**/*.key)",
      "Read(**/*.pem)",
      "Read(**/*.p12)",
      "Read(**/*.pfx)",
      "Read(**/id_rsa*)",
      "Read(**/credentials)",
      "Read(**/auth.json)",
      "Read(**/token*)",
      "Write(**/.env)",
      "Write(**/.env.local)",
      "Write(**/.env.development)",
      "Write(**/.env.production)",
      "Write(**/.env.staging)",
      "Write(**/.env.test)",
      "Write(**/secrets/**)",
      "Write(**/.aws/**)",
      "Write(**/.ssh/**)"
    ]
  },
  "env": {
    "DISABLE_AUTOUPDATER": "0",
    "DISABLE_TELEMETRY": "1",
    "DISABLE_ERROR_REPORTING": "1",
    "DISABLE_BUG_COMMAND": "1"
  }
}
```

**What each setting does:**

**Allow section:**

- **`WebFetch` / `WebSearch`**: Enables Claude to fetch documentation and search the web for current information
- **`node_modules/` / `vendor/`**: Allows reading dependencies for better debugging
- **`build/` / `dist/` etc.**: Allows reading build outputs for analysis
- **Lock files**: Allows reading package manager lock files for version debugging
- **Template files**: Allows reading/editing .example, .template, .sample files

**Ask section (prompts for approval):**

- **`.git/config`**: Prompts before reading/modifying git configuration
- **`git push`**: Prompts before pushing code
- **`rm -rf`**: Prompts before destructive delete commands
- **Publishing commands**: Prompts before npm/yarn/pnpm publish
- **Critical dependency files**: Prompts before modifying package.json, Cargo.toml, etc.

**Deny section (hard blocked):**

- **`.env` files**: Blocks all environment files except templates
- **`secrets/`**: Blocks secrets directories
- **Cloud credentials**: Blocks AWS, SSH, Docker, Kubernetes configs
- **Private keys**: Blocks all key formats (.key, .pem, .p12, .pfx, SSH keys)
- **Auth tokens**: Blocks credentials, auth.json, token files
- **Additional security**: Blocks .netrc, GPG keys, Google Cloud service accounts

**Environment variables:**

- **`DISABLE_AUTOUPDATER: "0"`**: Keeps auto-updates ENABLED for security patches
- **`DISABLE_TELEMETRY: "1"`**: Disables analytics/telemetry
- **`DISABLE_ERROR_REPORTING: "1"`**: Disables crash reporting
- **`DISABLE_BUG_COMMAND: "1"`**: Disables /bug command

---

## Step 3: Configure Project-Level Privacy Settings

These settings are **specific to each project** and can be shared with your team via git. Use `.claude/settings.json` to apply to the **whole project** and **all team members** working on the project and `.claude/settings.local.json` to **only apply** to **your machine**. 

> - Make sure to add `.claude/settings.local.json` to `.gitignore` so it doesn't override other team members settings. 
> 
> - Keep project settings minimal - just project-specific overrides.

### File Location

In your project's root directory:

```
your-project/
├── .claude/
│   └── settings.json       # Shared team settings (commit to git)
│   └── settings.local.json # Personal overrides (add to .gitignore)
```

### Creating Project Settings

**macOS / Linux / Windows:**

```bash
# From your project root
mkdir -p .claude
```

Create `.claude/settings.json`:

```json
{
  "permissions": {
    "allow": [
      // ← add Project-specific that differ from global
    ],
    "ask": [
      // ← add Project-specific that differ from global
    ],
    "deny": [
      // ← add Project-specific that differ from global
    ]
  },
  "companyAnnouncements": [
    "Privacy Mode Enabled: Sensitive files blocked, web access allowed for docs"
  ]
}
```

**Note**: You don't need to repeat rules from `~/.claude/settings.json`. The global settings automatically apply to all projects. Only add project-specific rules here that are unique to this project.

**When to use project settings:**
- Block additional project-specific files (e.g., `Read(./internal-docs/**)`)
- Override global allow rules for this project only
- Add project-specific company announcements

### Add to `.gitignore`

Ensure personal settings aren't committed:

```gitignore
# Claude Code personal settings
.claude/settings.local.json

# Claude Code state and cache
.claude.json
```

### Settings Precedence

Settings are applied in this order (highest to lowest priority):

1. **Enterprise managed policies** (if applicable)
2. **Command line arguments** (temporary)
3. **`.claude/settings.local.json`** (personal, not committed)
4. **`.claude/settings.json`** (team, committed to git)
5. **`~/.claude/settings.json`** (global user settings)

---

## Verification Checklist

Use this checklist to verify your privacy configuration:

### Account-Level

- [ ] Opted out of model training in claude.ai settings
- [ ] Confirmed data retention is 30 days (check email confirmation)

### User-Level Settings

- [ ] Created `~/.claude/settings.json`
- [ ] Verified `WebFetch` and `WebSearch` are in allow list
- [ ] Verified `node_modules/` and `vendor/` are in allow list
- [ ] Verified template files (*.example, *.template) are in allow list
- [ ] Verified ask mode is configured for git config and critical operations
- [ ] Verified all sensitive files (.env, secrets/, .aws/, .ssh/) are in deny list
- [ ] Verified `DISABLE_AUTOUPDATER` is set to `"0"` (enabled)
- [ ] Verified telemetry and error reporting are disabled

### Project-Level Settings

- [ ] Created `.claude/settings.json` in project root
- [ ] Added project-specific allow patterns (if applicable)
- [ ] Added project-specific ask patterns (if applicable)
- [ ] Added project-specific deny patterns (if applicable)
- [ ] Added `.claude/settings.local.json` to `.gitignore`
- [ ] Committed settings to git for team sharing

### Test the Configuration

**Test file blocking:**

```bash
# Start Claude Code
claude

# Test 1: Try to read a blocked file (should be denied)
# In Claude: "Can you read my .env file?"
# Expected: Claude should refuse or not see the file

# Test 2: Try to read an allowed template file (should work)
# In Claude: "Can you read .env.example?"
# Expected: Claude should read it successfully

# Test 3: Try to fetch a URL (should work)
# In Claude: "Can you fetch https://docs.python.org/3/"
# Expected: Claude should fetch and display content

# Test 4: Try a critical operation (should prompt)
# In Claude: "Can you run git push?"
# Expected: Claude should ask for permission first
```

**Verify env variables are set:**

```bash
# Check autoupdater setting in the config file
grep DISABLE_AUTOUPDATER ~/.claude/settings.json
# Should show "DISABLE_AUTOUPDATER": "0" or the line should not exist

# Note: Environment variables from settings.json are applied by Claude Code
# at startup, not exported to your shell. To verify they're working:
# 1. Check the settings file exists and is valid JSON
# 2. Start Claude Code and observe its behavior
# 3. Check for telemetry network requests (should be none)

# If you want to set these as shell environment variables too:
echo 'export DISABLE_TELEMETRY=1' >> ~/.zshrc  # or ~/.bashrc
echo 'export DISABLE_ERROR_REPORTING=1' >> ~/.zshrc
source ~/.zshrc
```

---

## What This Does (and Doesn't Do)

### ✅ What You're Protected From

1. **Sensitive file access**: Claude cannot read/write .env files, secrets, SSH keys, AWS credentials, private keys
2. **Long data retention**: Data kept for only 30 days (vs 5 years)
3. **Telemetry collection**: Statsig analytics disabled
4. **Error reporting**: Sentry crash reports disabled
5. **Accidental credential exposure**: Multiple layers of protection for secrets

### ✅ What You Still Have Access To

1. **Web access**: Can fetch documentation and search for current information
2. **Dependency debugging**: Can read node_modules/, vendor/ for better error analysis
3. **Build analysis**: Can read build outputs and compiled code
4. **Version debugging**: Can read lock files for dependency version info
5. **Security updates**: Auto-updates enabled for latest patches
6. **Controlled operations**: Ask mode prompts for critical actions (git push, publishing, etc.)

### ❌ What You're NOT Protected From

1. **Server-side processing**: Your code still goes through Anthropic's servers
2. **Prompt storage**: Prompts are still stored for 30 days for trust & safety
3. **Zero retention**: Can't achieve true zero retention on consumer accounts
4. **Local-only processing**: No offline/local LLM mode available
5. **Provider bypass**: Can't send requests directly to model without Anthropic infrastructure

### Comparison to Cursor Privacy Mode

|Feature|Cursor Privacy Mode|Claude Code (This Config)|
|---|---|---|
|Block sensitive files|✅ Yes|✅ Yes|
|Disable telemetry|✅ Yes|✅ Yes|
|Reduced retention|✅ Yes (30 days)|✅ Yes (30 days)|
|Web access for docs|✅ Yes|✅ Yes|
|Read dependencies|✅ Yes|✅ Yes|
|Auto-updates|✅ Yes|✅ Yes|
|Ask mode for critical ops|✅ Yes|✅ Yes|
|Zero server storage|✅ Yes (Ghost Mode)|❌ No|
|Local-only processing|✅ Yes (Ghost Mode)|❌ No|
|Privacy tiers|✅ Yes (3 modes)|⚠️ Partial (2 modes)|

---

## Troubleshooting

### Settings Not Taking Effect

**Problem**: Claude Code still accessing blocked files

**Solution**:

```bash
# Restart Claude Code completely
# Check settings file syntax
cat ~/.claude/settings.json | python -m json.tool

# Verify no syntax errors
```

### Permission Errors

**Problem**: Claude Code throws permission errors or asks for approval unexpectedly

**Solution**: Check file paths in permissions - they must be exact matches:

- `Read(./.env)` blocks `./.env` only
- `Read(./.env.local)` blocks `./.env.local` specifically (no wildcards needed)
- `Read(./secrets/**)` blocks everything in secrets directory
- Items in `ask` will prompt - this is expected behavior

### Can't Opt Out of Training

**Problem**: Toggle doesn't appear in settings

**Solution**:

- Ensure you're logged into a Personal account (Free/Pro/Max)
- Team/Enterprise accounts manage this at org level
- API users have training disabled automatically

### Telemetry Still Being Sent

**Problem**: Network traffic shows telemetry requests

**Solution**:

```bash
# Verify environment variables are loaded
echo $DISABLE_TELEMETRY

# If empty, add to shell profile
echo 'export DISABLE_TELEMETRY=1' >> ~/.zshrc  # or ~/.bashrc
source ~/.zshrc
```

### WebFetch Not Working

**Problem**: Claude can't fetch URLs

**Solution:** Verify `WebFetch` is in the `allow` section and not the `deny` section:

```json
{
  "permissions": {
    "allow": [
      "WebFetch"  // Add it here to allow
    ],
    "deny": [
      // Remove "WebFetch" from here
    ]
  }
}
```

**Or use ask mode:**

```json
{
  "permissions": {
    "ask": [
      "WebFetch"  // Prompt before each web fetch
    ]
  }
}
```

### Auto-Updates Not Working

**Problem**: Claude Code isn't auto-updating

**Solution**: Check your settings file has auto-updates ENABLED:

```bash
grep DISABLE_AUTOUPDATER ~/.claude/settings.json
```

Should show `"DISABLE_AUTOUPDATER": "0"` or the line shouldn't exist at all.

If set to `"1"`, change it to `"0"` or remove the line entirely.

### Common Gotchas

**Problem**: `.env` files in subdirectories aren't being blocked

**Solution**: Make sure you're using `**/` glob patterns:
```json
"deny": [
  "Read(**/.env)"  // ✅ Correct - blocks everywhere
]
```
Not:
```json
"deny": [
  "Read(./.env)"   // ❌ Wrong - only blocks root
]
```

**Problem**: Settings seem to conflict between global and project

**Solution**: Remember the hierarchy:
- `deny` rules from ALL levels combine (most restrictive)
- Higher precedence levels override lower for the same rule
- Project settings don't need to repeat global settings

---

## Team Rollout Guide

### For Team Leads

**1. Create a shared repository configuration:**

```bash
# In your project repo
mkdir -p .claude
cp /path/to/this/guide/settings.json .claude/settings.json
git add .claude/settings.json
git commit -m "Add Claude Code privacy configuration"
git push
```

**2. Share this guide with your team:**

Send this document and require all team members to:

- Complete Step 1 (opt out of training)
- Complete Step 2 (user-level settings)
- Pull the latest repo to get project settings

**3. Verify compliance:**

Create a simple script to check team compliance:

```bash
#!/bin/bash
# check-privacy.sh

echo "Checking Claude Code privacy configuration..."

# Check user settings exist
if [ -f ~/.claude/settings.json ]; then
    echo "✅ User settings found"
else
    echo "❌ User settings missing - run setup"
fi

# Check for DISABLE_TELEMETRY
if [ "$DISABLE_TELEMETRY" = "1" ]; then
    echo "✅ Telemetry disabled"
else
    echo "❌ Telemetry not disabled"
fi

# Check project settings
if [ -f .claude/settings.json ]; then
    echo "✅ Project settings found"
else
    echo "❌ Project settings missing"
fi
```

---

## Additional Resources

- **Official Documentation**: [https://code.claude.com/docs/en/settings](https://code.claude.com/docs/en/settings)
- **Privacy Policy**: [https://www.anthropic.com/legal/privacy](https://www.anthropic.com/legal/privacy)
- **Support**: [https://support.claude.com/](https://support.claude.com/)

---

## Summary

You've now configured Claude Code with **balanced privacy and functionality** settings. Remember:

1. ✅ **Opt out of training** in account settings (critical!)
2. ✅ **Configure user settings** in `~/.claude/settings.json`
3. ✅ **Configure project settings** in `.claude/settings.json`
4. ✅ **Share with team** via git

This configuration provides **strong security** (blocks all credentials and secrets) while maintaining **full functionality** (web access, dependency reading, auto-updates) and giving you **control** (ask mode for critical operations).

**Data retention after this configuration**: 30 days (vs 5 years default)

**Key benefits over maximum privacy mode**:

- ✅ Can fetch current documentation and web content
- ✅ Can debug dependencies effectively
- ✅ Gets automatic security patches
- ✅ Prompts for critical operations (git push, publishing)
- ✅ Still blocks all sensitive credentials

---

_Last updated: January 2026_ _Version: 2.0 - Balanced Privacy & Functionality_