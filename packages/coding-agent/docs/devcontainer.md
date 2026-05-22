# Devcontainer Setup

Pi works inside VS Code devcontainers and GitHub Codespaces. The main consideration is that subscription credentials (OAuth tokens) are stored on the host in `~/.pi/agent/auth.json` and are not automatically available inside the container.

## Mounting credentials from the host

The simplest approach is to mount the host pi config directory into the container by adding a `mounts` entry to `devcontainer.json`:

```json
{
  "mounts": [
    "source=${localEnv:HOME}/.pi/agent,target=/home/vscode/.pi/agent,type=bind,consistency=cached"
  ]
}
```

Adjust the `target` path to match the container user's home directory. For root:

```json
{
  "mounts": [
    "source=${localEnv:HOME}/.pi/agent,target=/root/.pi/agent,type=bind,consistency=cached"
  ]
}
```

This shares the auth file, settings, and sessions between the host and the container. Any `/login` or `/logout` performed inside the container also updates the host.

## Using a custom config directory

Set `PI_CODING_AGENT_DIR` to point pi at an arbitrary directory instead of `~/.pi/agent/`. Use this when you want a container-specific config directory or when the home directory is not easily mountable:

```json
{
  "containerEnv": {
    "PI_CODING_AGENT_DIR": "/workspace/.pi/agent"
  },
  "mounts": [
    "source=${localWorkspaceFolder}/.pi/agent,target=/workspace/.pi/agent,type=bind,consistency=cached"
  ]
}
```

With this setup the config directory lives inside the repo checkout, which Codespaces or Docker already mounts.

## Logging in from inside the container

Pi displays the OAuth login URL as a clickable hyperlink. In devcontainers, `xdg-open` usually cannot launch a browser, but the URL is still visible in the terminal. To complete login:

1. Run `/login` inside pi and select a provider.
2. Ctrl+click the URL in the terminal (supported by most terminals), or copy and paste it into a host browser.
3. Complete the OAuth flow in the browser.

For **GitHub Copilot**, the device code flow shows a URL and a short user code to enter at the URL — no automatic browser launch is required. This flow works inside devcontainers without any extra setup beyond credential sharing.

## Codespaces

GitHub Codespaces mounts the workspace but not the host home directory. Credentials must come from one of:

- A devcontainer `mounts` entry pointing at a path available in the codespace (not applicable for cloud-hosted codespaces with no host filesystem).
- An environment variable set as a [Codespaces secret](https://docs.github.com/en/codespaces/setting-your-account-level-secret):

  ```
  ANTHROPIC_API_KEY=sk-ant-...
  ```

  Pi resolves environment-variable API keys as a fallback when no auth.json entry is present.

- Log in from inside the codespace. The URL will appear in the integrated terminal. Copy it and open it in the host browser to complete the OAuth flow. Credentials are written to `~/.pi/agent/auth.json` inside the codespace and persist for the life of that codespace.
