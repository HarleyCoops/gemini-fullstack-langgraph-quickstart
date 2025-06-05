# OpenAI Codex Sandbox Environment Setup (PhD Level)

This guide provides a detailed, in-depth walkthrough for setting up the environment for OpenAI's Codex, focusing on understanding and resolving common sandbox-related issues. Codex operates within isolated micro-VM sandboxes, which are critical for security and consistent execution. Misconfigurations in this environment are a frequent cause of "sandbox unavailable" or "environment variable ignored" errors.

## 1. Introduction to OpenAI Codex and its Sandbox

OpenAI Codex is a powerful coding agent designed to run in your terminal, capable of understanding natural language and generating/modifying code. A core aspect of its operation is the use of **micro-VM sandboxes**.

Each time you initiate a task with Codex (e.g., by pressing "Run" in a sidebar or executing a command), a new, isolated micro-VM sandbox is provisioned. This sandbox includes:
*   Its own file system
*   Dedicated CPU and RAM resources
*   A locked-down network policy
*   A clone of your repository
*   Pre-installed common developer tools (linters, formatters, test runners)

**Why the Sandbox is Critical:**
*   **Security:** Prevents the agent from making arbitrary, potentially malicious API calls or accessing unauthorized resources outside its designated environment.
*   **Isolation:** Ensures that each execution is clean and unaffected by previous runs or the host system's state, providing consistent results.
*   **Reproducibility:** Guarantees that the environment for code execution is standardized, minimizing "it works on my machine" scenarios.

When the sandbox fails to initialize, is unavailable, or cannot connect, Codex cannot proceed safely or correctly, leading to a critical error and process termination (e.g., "Sandbox was mandated, but no sandbox is available!").

## 2. Prerequisites

Before you begin, ensure you have the following:

*   **Node.js and npm:** Codex is an npm package.
    *   Verify installation: `node -v` and `npm -v`
    *   If not installed, download from [nodejs.org](https://nodejs.org/).
*   **OpenAI API Key:** You need an API key from OpenAI to use Codex. Obtain one from the [OpenAI API platform](https://platform.openai.com/account/api-keys).

## 3. Global Installation of OpenAI Codex CLI

Install the Codex command-line interface (CLI) globally on your system. This allows you to run Codex commands from any directory.

```bash
npm install -g @openai/codex
```

## 4. Setting Up Environment Variables for Codex CLI

The Codex CLI requires your OpenAI API key to authenticate with the OpenAI services. This key should be set as an environment variable.

### For Linux/macOS (Bash/Zsh):

To set the `OPENAI_API_KEY` temporarily for the current terminal session:

```bash
export OPENAI_API_KEY="your-api-key-here"
```

To set it permanently (recommended), add the `export` command to your shell's profile file (e.g., `~/.bashrc`, `~/.zshrc`, or `~/.profile`):

1.  Open your profile file in a text editor:
    ```bash
    nano ~/.bashrc # or ~/.zshrc, ~/.profile
    ```
2.  Add the following line to the end of the file, replacing `"your-api-key-here"` with your actual key:
    ```bash
    export OPENAI_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
    ```
3.  Save the file and exit the editor.
4.  Apply the changes by sourcing the file:
    ```bash
    source ~/.bashrc # or ~/.zshrc, ~/.profile
    ```
    You can verify it's set by running `echo $OPENAI_API_KEY`.

### For Windows (Command Prompt):

To set the `OPENAI_API_KEY` temporarily for the current command prompt session:

```cmd
set OPENAI_API_KEY="your-api-key-here"
```

To set it permanently:

1.  Open the System Properties dialog: Press `Win + R`, type `sysdm.cpl`, and press Enter.
2.  Go to the "Advanced" tab and click "Environment Variables...".
3.  Under "User variables for [Your Username]" (or "System variables" if you want it available for all users), click "New...".
4.  For "Variable name", enter `OPENAI_API_KEY`.
5.  For "Variable value", enter your actual API key (e.g., `sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx`).
6.  Click "OK" on all dialogs to save the changes.
7.  Restart any open Command Prompt or PowerShell windows for the changes to take effect.

### For Windows (PowerShell):

To set the `OPENAI_API_KEY` temporarily for the current PowerShell session:

```powershell
$env:OPENAI_API_KEY="your-api-key-here"
```

To set it permanently, you can add it to your PowerShell profile. First, check if you have a profile:

```powershell
Test-Path $PROFILE
```
If it returns `False`, create one:
```powershell
New-Item -Path $PROFILE -ItemType File -Force
```
Then, open the profile and add the environment variable:
```powershell
notepad $PROFILE
```
Add the following line, replacing `"your-api-key-here"` with your actual key:
```powershell
$env:OPENAI_API_KEY="sk-xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
```
Save the file and restart PowerShell.

## 5. Understanding and Configuring the Codex Sandbox Environment

This is where the "PhD level" understanding comes in. While you set `OPENAI_API_KEY` for the *CLI*, the **Codex micro-VM sandbox has its own environment**.

**Key Insight:** Environment variables and setup commands for the *sandbox itself* are typically configured through the **Codex web application's "Environment setup" or "Advanced settings/environment"**. They are **NOT** usually read from `.env` files within your cloned repository inside the sandbox.

*   **Isolation Principle:** The sandbox is designed to be isolated. This means that local environment variables on your machine, or `.env` files in your project, are generally *not* automatically injected into the sandbox's runtime environment.
*   **Web App Configuration:** When using Codex through a web interface (e.g., Coding with ChatGPT, or similar platforms that host Codex), you will find a dedicated section to "Configure the environment." This is where you add environment variables, secrets, or setup commands (like `apt-get update` or installing specific dependencies) that will be executed *within* the micro-VM before your code runs.
    *   **Example:** If your project needs a specific `DATABASE_URL` or a custom `NPM_TOKEN`, you would add these directly in the Codex web app's environment configuration, not just in your local `.env` file.
*   **Setup Commands:** Similarly, if your sandbox needs to install additional packages (e.g., `pip install pandas` or `npm install`), these commands should be specified in the "setup commands" section of the web app's environment configuration.
    *   **Common Pitfall:** Avoid using `sudo` in these setup commands (e.g., `sudo apt-get update`). The sandbox environment often runs as root, so `sudo` is unnecessary and can sometimes cause permission errors or unexpected behavior. Use `apt-get update` directly.

## 6. Troubleshooting Common Sandbox Errors

### "Sandbox was mandated, but no sandbox is available!" or "Failed to set up container"

*   **API Key Issues:** Double-check that your `OPENAI_API_KEY` is correctly set and active. An invalid or missing key can prevent the sandbox from being provisioned.
*   **Network Configuration:** If you are self-hosting or have complex network setups, ensure that Codex can properly determine its public IP address. You might need to manually adjust it using the `--nat` CLI argument or the `CODEX_NAT` environment variable.
*   **Resource Limitations:** Ensure your system has sufficient resources (CPU, RAM, disk space) to spin up the micro-VM sandbox.
*   **Firewall/Proxy:** Check if firewalls or proxies are blocking communication required for sandbox initialization.
*   **Codex Web App Environment:** If using a web-based Codex, ensure all necessary environment variables and setup commands are correctly configured *within the web app's environment settings*. Do not rely solely on local `.env` files.

### "Custom environment variables are ignored"

*   **Web App Configuration is Key:** As highlighted in Section 5, custom environment variables for the sandbox must be configured directly within the Codex web application's environment setup. They are not typically inherited from your local machine's environment or `.env` files in your repository.
*   **Advanced Scripting:** Some Codex environments might allow "advanced scripts" for setup. Ensure your variables are correctly exposed within these scripts if you are using them.

### "Codex js sandbox: refused to connect" (for external APIs)

*   **Security Policy:** By design, the Codex sandbox often has a locked-down network policy that prevents calls to arbitrary external APIs for security reasons. If your code needs to interact with external services, you might need to:
    *   Export the code to a different environment (e.g., JSFiddle, your own webpage) where you have full network control.
    *   Check if the specific Codex platform you are using offers a secure way to whitelist or proxy external API calls.

## 7. Testing Your Codex Environment

To test your setup, you can try a simple task with the Codex CLI:

1.  Navigate to a simple code repository (e.g., a small Python or JavaScript project).
2.  Run a basic Codex command, for example:
    ```bash
    codex "refactor this code to be more readable"
    ```
    or
    ```bash
    codex "write a simple 'hello world' function in Python"
    ```
    Observe the output for any sandbox-related errors. If it runs successfully, your environment is likely configured correctly.

This comprehensive guide should help you diagnose and resolve most environment and sandbox-related issues when working with OpenAI Codex.
