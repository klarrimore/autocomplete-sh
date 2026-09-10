Autocomplete.sh
========================================================

AI command suggestions for your terminal, without taking over Tab.

Autocomplete.sh keeps normal shell completion fast and local. Press <kbd>Tab</kbd>
and Bash or Zsh completes commands, flags, files, and paths the way it already
does. When you want help turning an idea into a command, ask explicitly:

```bash
autocomplete command "list the files here, largest first"
```

![Autocomplete.sh Demo](https://github.com/user-attachments/assets/6f2a8f81-49b7-46e9-8005-c8a9dd3fc033)

## Quick Start

Make sure `$HOME/.local/bin` is on your `PATH`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

Download and run the installer:

```bash
wget -O install.sh https://autocomplete.sh/install.sh
bash install.sh --shell bash --version main
```

For Zsh:

```bash
bash install.sh --shell zsh --version main
```

Reload your shell:

```bash
source ~/.bashrc
# or
source ~/.zshrc
```

Choose a model and provider:

```bash
autocomplete model
```

Then ask for your first command:

```bash
autocomplete command "find large log files modified today"
```

Autocomplete.sh prints numbered suggestions. It does not execute them for you.

## What Tab Does

Tab stays shell-native.

- <kbd>Tab</kbd> uses Bash or Zsh completion, not an LLM.
- Tab completion works offline and does not create provider requests.
- Existing completion definitions from tools such as Git, Docker, Mise, and
  package managers remain responsible for their own flags and arguments.
- On an empty Bash prompt, Tab does not expand to every command on the system.

Bash users can choose how ambiguous native completions display:

```bash
autocomplete config set tab_display_mode list   # default Bash behavior
autocomplete config set tab_display_mode menu   # cycle choices inline on Tab
source autocomplete enable
```

`menu` can feel calmer when a command has many possible flags. It still uses
native Bash completion. It does not call an AI provider.

## Ask AI Explicitly

Use `autocomplete command` when you know what you want to do but not the exact
syntax:

```bash
autocomplete command "compress this directory into a dated tar.gz"
autocomplete command "show listening ports with process names"
autocomplete command "create a git branch for fixing login redirects"
```

Preview the exact prompt without making a provider request:

```bash
autocomplete command --dry-run "restart nginx safely"
```

## Bash Live-Line Actions

Bash also includes AI actions for the current command line. They print numbered
suggestions and never execute output.

```bash
autocomplete ai-complete "git ch"
autocomplete ai-rewrite "show failed units"
autocomplete context --mode ai-completion "git ch"
```

Default Readline bindings:

| Action | Binding | Command |
|---|---:|---|
| Complete the current line | <kbd>Ctrl-X Ctrl-A</kbd> | `autocomplete ai-complete` |
| Rewrite the current line | <kbd>Ctrl-X Ctrl-B</kbd> | `autocomplete ai-rewrite` |

To change or disable these bindings, set the variables before sourcing the
runtime:

```bash
export ACSH_AI_COMPLETE_KEY='\C-x\C-a'
export ACSH_AI_REWRITE_KEY='\C-x\C-b'
source autocomplete enable
```

Set either variable to an empty string to disable that binding.

Zsh users can use the cross-shell command interface:

```bash
autocomplete command "your request here"
```

## Configuration

View the active configuration:

```bash
autocomplete config
```

If your shell is enabled, the status line should show `Enabled`. If it does
not, reload the shell integration:

```bash
source autocomplete enable
```

Update a setting:

```bash
autocomplete config set <key> <value>
```

Common settings:

| Setting | Default | Purpose |
|---|---:|---|
| `provider` | `openai` | LLM provider to use for explicit AI requests |
| `model` | provider default | Model name sent to the provider |
| `temperature` | `0.0` | Controls variation in suggestions |
| `request_timeout_seconds` | `5` | Maximum time for one provider request |
| `cache_size` | `10` | Number of cached AI answers to keep |
| `tab_display_mode` | `list` | Bash native completion display: `list` or `menu` |

![Configuration Options](https://github.com/user-attachments/assets/61578f27-594f-4bc4-ba86-c5f99a41e8a9)

## Providers

Autocomplete.sh supports:

- OpenAI
- Anthropic
- Groq
- Ollama
- Any OpenAI-compatible Chat Completions endpoint

The easiest setup path is:

```bash
autocomplete model
```

For a local or self-hosted OpenAI-compatible endpoint:

```bash
autocomplete config set provider openai-compatible
autocomplete config set endpoint http://localhost:8080/v1/chat/completions
autocomplete config set model your-model-name
autocomplete config set request_timeout_seconds 5
autocomplete config set extra_body_json '{"chat_template_kwargs":{"enable_thinking":false}}'
```

If `openai_compatible_api_key` is empty, Autocomplete.sh omits the
`Authorization` header. This works for keyless local endpoints. To send a key,
set it in config or export `OPENAI_COMPATIBLE_API_KEY`.

```bash
autocomplete config set openai_compatible_api_key sk-your-key
```

Provider notes:

- `request_headers_json` and `extra_body_json` only work with
  `openai-compatible`.
- `Authorization` and `Content-Type` are managed by Autocomplete.sh and cannot
  be overridden in `request_headers_json`.
- Reserved request-body keys are rejected: `model`, `messages`, `tools`,
  `tool_choice`, `response_format`, and `stream`.

![Model Selection](https://github.com/user-attachments/assets/6206963f-81c2-4d68-b054-6ec88969ba0c)

## Context and Privacy

By default, AI requests include only:

- the mode
- your active shell
- your input
- output instructions

Extra context is opt-in. Enable only the sections you want:

| Key | Default | Included when enabled |
|---|---:|---|
| `context_terminal` | `false` | Working directory, OS type, shell, terminal type |
| `context_environment` | `false` | Environment variable names only, never values |
| `context_history` | `false` | Recent commands with secrets redacted |
| `context_recent_files` | `false` | Current-directory file basenames only |
| `context_help` | `false` | Bounded `--help` output for the first executable |

Context limits:

| Setting | Default |
|---|---:|
| `max_environment_names` | `50` |
| `max_history_commands` | `10` |
| `max_recent_files` | `10` |
| `max_help_lines` | `40` |
| `help_timeout_seconds` | `0.25` |

See what would be sent before making a request:

```bash
autocomplete config set context_history true
autocomplete command --dry-run "undo my last git commit but keep the changes"
```

## Usage and Cache

Show request usage:

```bash
autocomplete usage
```

Clear cached answers and logs:

```bash
autocomplete clear
```

![Usage Statistics](https://github.com/user-attachments/assets/0fc611b9-fb4c-4f68-bf01-8e6ecdcf7410)

## Development

Install a local checkout:

```bash
git clone git@github.com:closedloop-technologies/autocomplete-sh.git
cd autocomplete-sh
./docs/install.sh --shell bash --version dev
```

For Zsh:

```bash
./docs/install.sh --shell zsh --version dev
```

Run the tests:

```bash
sudo apt install bats shellcheck zsh
bats tests
```

Run the Docker test image:

```bash
docker build -t autocomplete-sh .
docker run --rm autocomplete-sh
```

## Maintainers

Maintained by Sean Kruzel [@closedloop](https://github.com/closedloop) at
[Closedloop.tech](https://Closedloop.tech).

Contributions and bug fixes are welcome.

## Support Open Source

The best way to support Autocomplete.sh is to use it, share it, and star the
project.

- [Just use it](https://github.com/closedloop-technologies/autocomplete-sh?tab=readme-ov-file#quick-start)
- [Share it](https://x.com/intent/post?text=I+love+autocomplete.sh%21++%22autocomplete+command%22+helps+me+build+quickly+%40JustBuild_ai)
- Star it

If you want to support ongoing work:

[!["Buy Me A Coffee"](https://www.buymeacoffee.com/assets/img/custom_images/orange_img.png)](https://www.buymeacoffee.com/skruzel)

## License

See the [MIT license](./LICENSE).
