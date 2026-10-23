# STYX Shell

STYX Shell is a single-file, browser-based terminal (web shell) written in PHP. It provides an interactive command-line interface over HTTP, including tab-completion, command history, working-directory persistence, and file upload/download — all from a single `styx.php` file with no external dependencies.

## Features

- Fully interactive browser terminal (no frameworks, no build step required).
- Working directory (`cwd`) is preserved across requests, including via `cd`.
- Tab completion: command names after the prompt, file names after a command.
- Command history via **Arrow Up / Arrow Down**.
- Built-in file upload (`upload <path>`) and download (`download <path>`).
- Busy-state protection: while a long-running command is executing, the input is disabled and marked with a blinking `running…` indicator, preventing accidental command stacking.
- Optional HTTP Basic authentication.
- UTF-8-safe output rendering.

## Requirements

- PHP 7.0 or newer (PHP 8.x supported and recommended).
- A web server (Apache, nginx with PHP-FPM, or PHP's built-in server for local testing).
- At least one of the following functions enabled: `exec`, `shell_exec`, `system`, `passthru`, `popen`, or `proc_open`. The script uses whichever is available, in that order of preference.

## Installation

1. Place `styx.php` in a directory served by your web server (or use PHP's built-in server locally):

   ```bash
   php -S 127.0.0.1:8080 styx.php
   ```

2. Open the URL in a browser. The terminal initializes and displays the prompt `user@host:~#`.

3. **Before deploying anywhere reachable by others, enable authentication** (see *Configuration* and *Security Notice* below). The script intentionally ships with authentication disabled so that it works out of the box in a local, sandboxed environment.

## Configuration

All configuration lives in the `$SHELL_CONFIG` array at the top of `styx.php`:

```php
$SHELL_CONFIG = array(
    'username' => 'styx',
    'hostname' => 'shell',
    'password' => '', // set non-empty to enable HTTP Basic auth
);
```

| Key        | Purpose                                                                    |
|------------|----------------------------------------------------------------------------|
| `username` | Fallback display name. Overridden automatically by the running OS user.    |
| `hostname` | Fallback display name. Overridden automatically by the machine's hostname. |
| `password` | Plain text password for HTTP Basic authentication. Empty string disables authentication. |

Example hardening:

```php
'password' => 'a-long-random-secret-here'
```

When a password is set, the browser will prompt once; all subsequent page and XHR requests are authenticated automatically.

## Usage

| Action                    | How                                                              |
|---------------------------|------------------------------------------------------------------|
| Run a command             | Type it and press **Enter** (stderr is merged into stdout automatically). |
| Change directory          | `cd /var/log` — supports paths with spaces (`cd my folder`), quoted paths (`cd "my folder"`), and `~` / `~user` expansion. |
| Tab completion            | **Tab** completes a command name; after a command, completes file/folder names. Multiple candidates are listed. |
| Command history           | **Arrow Up / Arrow Down** cycles through previous commands.       |
| Download a file           | `download /path/to/file` — the browser saves it to your download directory. |
| Upload a file             | `upload /tmp/target-name` — a file picker opens; the file is saved at the given path. |
| Clear the screen          | `clear`                                                           |

## Caution — Read Before Deploying

**This tool is effectively a remote code-execution backdoor for any host it runs on.** Handle it strictly as a privileged utility.

1. **Never deploy it to a publicly reachable server without authentication.** Set a strong, unique password in `$SHELL_CONFIG['password']` before exposing the file beyond `localhost`.
2. **Prefer keeping it on an internal or loopback-only endpoint**, protected by network-level controls (firewall rules, VPN, or restrictive `Allow/Deny` directives) in addition to the built-in authentication.
3. **Do not commit real credentials** to any repository. The example password must be replaced with your own secret at deployment time.
4. **Delete it after use** if it was uploaded temporarily for troubleshooting a machine. An unguarded leftover shell is a common Post-Exploitation / persistence artifact and a serious Indication of Compromise.
5. **Expect fewer hardened environments to block it.** Hosting providers frequently disable `exec`-family functions precisely to defeat tools of this kind; if output is empty, verify your PHP configuration (`disable_functions`).
6. **HTTP Basic authentication transmits credentials per request** (Base64-encoded, not encrypted). Always serve this tool exclusively over HTTPS in real environments.
7. **You are fully responsible** for any command executed through this interface. There is no sandboxing, auditing, or rate limiting; commands run with the privileges of the PHP process user.
8. Use this tool **only on systems you own or are explicitly authorized to administer.** Unauthorized use against any other system is illegal in most jurisdictions.
