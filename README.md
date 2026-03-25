# git-ai-commit

A bash CLI tool that generates commit messages using AI based on git diff analysis.

## Features

- **AI-powered commit messages**: Uses Claude or OpenAI API to generate conventional commit messages
- **Flexible diff analysis**: Analyze all changes or only staged changes
- **Dry run mode**: Preview commit messages without committing
- **Customizable context**: Include more or less context in the analysis
- **Multiple AI providers**: Support for Claude, OpenAI (default), and Google Gemini
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
# Basic usage - analyze all changes and commit
git-ai-commit

# Only analyze staged changes
git-ai-commit --staged

# Preview commit message without committing
git-ai-commit --dry-run

# Include more context in analysis
git-ai-commit --context

# Limit diff analysis to 200 lines
git-ai-commit --limit 200

# Use OpenAI instead of Claude
git-ai-commit --provider openai

# Use Google Gemini
git-ai-commit --provider gemini
```

## Options

- `-s, --staged`: Only analyze staged changes (default: all changes)
- `-c, --context`: Include more context in the analysis
- `-d, --dry-run`: Show the commit message without committing
- `-p, --provider`: AI provider to use (claude, openai, gemini) [default: openai]
- `-l, --limit`: Maximum number of diff lines to analyze [default: 500]
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
feat(cli): add AI-powered commit message generation tool

Dry run mode - not committing
```

## How it Works

1. **Analyzes git diff**: Captures changes using `git diff` or `git diff --staged`
2. **Sends to AI API**: Makes HTTP request to Claude or OpenAI API with the diff and prompt
3. **Generates message**: AI analyzes the changes and creates an appropriate commit message
4. **Confirms with user**: Shows the generated message and asks for confirmation
5. **Creates commit**: If approved, stages changes (if needed) and creates the commit

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
