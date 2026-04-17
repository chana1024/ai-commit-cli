# git-ai-commit

A bash CLI tool that generates commit messages using AI based on structured git diff analysis.

## Features

- **AI-powered commit messages**: Uses Claude, OpenAI, or Gemini to generate conventional commit messages with subject + body
- **Staged-first workflow**: Analyze and commit only staged changes by default
- **Explicit `--all` mode**: Stage all changes first when you want the tool to include everything
- **Dry run mode**: Preview commit messages without committing
- **Customizable context**: Include more or less context in the analysis
- **Multiple AI providers**: Support for Claude, OpenAI (default), and Google Gemini
- **Large diff coverage**: Includes `git diff --stat`, `git diff --name-status`, and per-file excerpts when the patch is large
- **Safety features**: Confirmation prompts and error handling

## Requirements

- Git repository
- `curl` or `wget` for HTTP requests (pre-installed on most systems)
- `jq` for JSON parsing (install with: `sudo apt install jq` or `brew install jq`)
- API key for your chosen AI provider
- Active git changes to analyze

## Installation

1. Download and move the script:
```bash
curl -O https://raw.githubusercontent.com/chana1024/ai-commit-cli/refs/heads/master/git-ai-commit
chmod +x git-ai-commit
sudo mv git-ai-commit /opt/bin/
```

2. The script will auto-create `~/.ai-commit-cli/config.json` on first run.

Default content:

```json
{
  "provider": "openai",
  "model": "gpt-4",
  "base_url": "https://api.openai.com",
  "api_key": ""
}
```

Fill in `api_key` before first real use, or provide `OPENAI_API_KEY` from the environment.
If your OpenAI-compatible provider does not require a key, you can leave it empty.

You can also edit it into per-provider settings:

```json
{
  "provider": "openai",
  "providers": {
    "openai": {
      "api_key": "your-openai-compatible-key",
      "model": "deepseek-chat",
      "base_url": "https://api.deepseek.com"
    }
  }
}
```

3. Or set up your API key with environment variables if you do not want it in the config file:
```bash
# For Claude
export ANTHROPIC_API_KEY="your-claude-api-key"

# For OpenAI
export OPENAI_API_KEY="your-openai-api-key"

# For Google Gemini
export GEMINI_API_KEY="your-gemini-api-key"

# Add to your shell profile (~/.bashrc, ~/.zshrc) to persist
echo 'export ANTHROPIC_API_KEY="your-claude-api-key"' >> ~/.bashrc
```

## Usage

```bash
# Basic usage - analyze staged changes and commit
git-ai-commit

# Explicit staged mode (same as default, kept for compatibility)
git-ai-commit --staged

# Stage all changes first, then analyze and commit them
git-ai-commit --all

# Preview commit message without committing
git-ai-commit --dry-run

# Include more context in analysis
git-ai-commit --context

# Limit large-diff patch excerpts to 200 lines
git-ai-commit --limit 200

# Use OpenAI instead of Claude
git-ai-commit --provider openai

# Use Google Gemini
git-ai-commit --provider gemini
```

## Options

- `-s, --staged`: Analyze staged changes (default behavior; kept for compatibility)
- `-a, --all`: Stage all changes before analysis and commit
- `-c, --context`: Include more context in the analysis
- `-d, --dry-run`: Show the commit message without committing
- `-p, --provider`: AI provider to use (claude, openai, gemini) [default: openai]
- `-l, --limit`: Maximum per-run patch excerpt lines for large diffs [default: 500]
- `-h, --help`: Show help message

## Configuration

Configuration priority is:

`command line > ~/.ai-commit-cli/config.json > environment variables > built-in defaults`

The config file supports these fields:

- `provider`
- `api_key`
- `model`
- `base_url`
- `providers.<provider>.api_key`
- `providers.<provider>.model`
- `providers.<provider>.base_url`

### Environment Variables

- `ANTHROPIC_API_KEY`: Required for Claude provider
- `OPENAI_API_KEY`: Optional for OpenAI provider. Leave empty for compatible providers that do not require auth.
- `GEMINI_API_KEY`: Required for Google Gemini provider
- `GIT_AI_COMMIT_MODEL`: Override default model (optional)
  - Claude: `claude-3-sonnet-20240229` (default) or `claude-3-haiku-20240307`
  - OpenAI: `gpt-4` (default) or `gpt-3.5-turbo`
  - Gemini: `gemini-1.5-flash` (default) or `gemini-1.5-pro` or `gemini-2.5-flash`
- `CLAUDE_API_URL`: Override Claude API URL or base URL (default endpoint: `https://api.anthropic.com/v1/messages`)
- `OPENAI_API_URL`: Override OpenAI API URL or base URL (default endpoint: `https://api.openai.com/v1/chat/completions`)
- `GEMINI_API_URL`: Override Gemini API URL or base URL (default root: `https://generativelanguage.googleapis.com/v1beta`)
- `DEBUG`: Enable debug logging (set to `true` or `1` to enable)

For `openai` provider, `base_url` may be either a full endpoint like
`https://api.openai.com/v1/chat/completions`, a provider root like
`https://api.openai.com`, or a versioned root like `https://api.deepseek.com/v1`.
The script normalizes it to `/v1/chat/completions` automatically.

### Example Configuration

```bash
# Use Claude Haiku for faster/cheaper responses
export GIT_AI_COMMIT_MODEL="claude-3-haiku-20240307"

# Use GPT-3.5 Turbo for OpenAI
export GIT_AI_COMMIT_MODEL="gpt-3.5-turbo"
git-ai-commit --provider openai

# Use Gemini Pro for better quality
export GIT_AI_COMMIT_MODEL="gemini-1.5-pro"
git-ai-commit --provider gemini

# Use latest Gemini 2.5 Flash (if available)
export GIT_AI_COMMIT_MODEL="gemini-2.5-flash"
git-ai-commit --provider gemini

# Use custom API endpoints or base URLs
export CLAUDE_API_URL="https://your-proxy.com"
export OPENAI_API_URL="https://your-proxy.com/v1"
export GEMINI_API_URL="https://your-proxy.com/v1beta"

# Enable debug logging for troubleshooting
export DEBUG=true
git-ai-commit --dry-run
```

### OpenAI-Compatible Provider Example

```json
{
  "provider": "openai",
  "providers": {
    "openai": {
      "api_key": "your-provider-key",
      "model": "deepseek-chat",
      "base_url": "https://api.deepseek.com"
    }
  }
}
```

## Example Output

```bash
$ git-ai-commit --dry-run
Info: Checking git repository...
Info: Checking dependencies...
Info: Analyzing changes...
Info: Generating commit message using claude...

Generated commit message:
feat(cli): improve structured commit message generation

- expand diff analysis with --stat, --name-status, and per-file excerpts for large patches [git-ai-commit]
- require a conventional commit subject plus body bullets that cover major change groups [git-ai-commit]
- preserve multi-line output in dry runs and git commits and document the new format [git-ai-commit, README.md]

Dry run mode - not committing
```

## How it Works

1. **Builds structured change context**: Captures `git diff --stat`, `git diff --name-status`, and either the full patch or per-file excerpts
2. **Sends to AI API**: Makes HTTP request to Claude, OpenAI, or Gemini with a strict subject + body prompt
3. **Generates message**: AI analyzes the changes and creates a conventional commit subject plus body bullets
4. **Confirms with user**: Shows the generated message and asks for confirmation
5. **Creates commit**: If approved, creates the commit from the current staged snapshot with the full multi-line message

`--all` stages changes before analysis so the analyzed snapshot matches the eventual commit. If you cancel after previewing, the staged changes remain in the index.

## Troubleshooting

### Common Issues

1. **"jq: command not found"**
   ```bash
   # Ubuntu/Debian
   sudo apt install jq
   
   # macOS
   brew install jq
   
   # CentOS/RHEL
   sudo yum install jq
   ```

2. **"API key not found"**
   - Ensure you've set the correct environment variable
   - Check that the API key is valid and active
   - Verify the variable is exported in your current shell session

3. **"Failed to parse API response"**
   - Check your internet connection
   - Verify the API key has sufficient credits/quota
   - Try again as the API might be temporarily unavailable
   - For Gemini: Ensure your API key has the Generative Language API enabled

4. **"Unsupported AI provider"**
   - Check that you're using a supported provider: claude, openai, or gemini
   - Verify the provider name is spelled correctly

### Debug Mode

Enable detailed debug logging to troubleshoot issues:

```bash
# Enable debug mode
export DEBUG=true
git-ai-commit --dry-run

# Or enable for a single run
DEBUG=1 git-ai-commit --provider gemini --dry-run
```

Debug output includes:
- Configuration validation
- Git diff analysis details
- Complete HTTP request/response information:
  - Request URL and headers
  - Full request body (pretty-printed JSON)
  - Response body (pretty-printed JSON, truncated if large)
  - Response length and parsing status
- Internal processing steps
- Error context and details

Add `-x` to the shebang line to enable debug output:
```bash
#!/bin/bash -x
```
