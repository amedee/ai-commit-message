# AI Commit Message Generator

A GitHub Action that generates concise, AI-powered Git commit messages using [OpenRouter](https://openrouter.ai/).

The action accepts a Git diff and a prompt, sends them to an OpenRouter model, and returns the generated commit message as an output.

It supports multiple models with automatic fallback and retry handling for transient OpenRouter errors.

## Usage

```yaml
- name: Generate commit message
  id: commit-message
  uses: amedee/ai-commit-message@v1
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
  with:
    diff_content: ${{ steps.diff.outputs.diff }}
    prompt_content: |
      Generate a concise commit message describing these changes.
      Use conventional commit style where appropriate.
```

The generated commit message is available through the `message` output:

```yaml
- name: Use commit message
  run: |
    echo "${{ steps.commit-message.outputs.message }}"
```

## Inputs

### `diff_content`

**Required:** Yes

The Git diff to analyze.

The action does not generate the diff itself. The calling workflow is responsible for obtaining and providing it.

### `prompt_content`

**Required:** Yes

Additional instructions describing what the AI should do with the supplied diff.

The action automatically appends its own commit-message formatting and style instructions to this prompt.

The generated message is instructed to:

* Start with a relevant emoji.
* Keep the first line to a maximum of 72 characters.
* Not end the first line with punctuation.
* Leave a blank line after the first line.
* Keep subsequent lines to a maximum of 120 characters.
* Allow Markdown formatting.
* Use a sassy and friendly tone.
* Refer to the project as a single-developer project, avoiding plural "we" and "our".
* Output only the commit message without explanations or reasoning.

### `models`

**Required:** No

**Default:**

```text
nvidia/nemotron-3-ultra-550b-a55b:free,
poolside/laguna-m.1:free,
cohere/north-mini-code:free,
openai/gpt-oss-120b:free,
google/gemma-4-31b-it:free,
liquid/lfm-2.5-1.2b-thinking:free
```

A comma-separated list of [OpenRouter](https://openrouter.ai/models) model identifiers.

Models are tried in the order specified. If a model fails, the action continues with the next model.

For example:

```yaml
models: |
  openai/gpt-oss-120b:free,
  cohere/north-mini-code:free
```

This can be useful when some models are temporarily unavailable or rate-limited.

## Outputs

### `message`

The generated and sanitized commit message.

Example:

```yaml
- name: Generate commit message
  id: commit-message
  uses: amedee/ai-commit-message@v1
  env:
    OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
  with:
    diff_content: ${{ steps.diff.outputs.diff }}
    prompt_content: |
      Generate a concise commit message for the supplied Git diff.
      Follow conventional commit conventions where appropriate.
      Return only the commit message.

- name: Display commit message
  run: |
    echo "${{ steps.commit-message.outputs.message }}"
```

## Authentication

The action uses the OpenRouter API and expects an API key in the `OPENROUTER_API_KEY` environment variable.

Store the API key as a GitHub Actions secret:

```yaml
env:
  OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

Do not put the API key directly in a workflow file.

## Error handling and retries

The action provides two levels of resilience when communicating with OpenRouter.

### Model fallback

Models are attempted sequentially.

If a model fails, the action logs a warning and continues with the next configured model.

If every model fails, the action fails with the error from the last attempted model.

### Transient API errors

HTTP `429 Too Many Requests` and `503 Service Unavailable` responses are retried with exponential backoff.

The action makes up to five attempts per model.

The initial retry delay is 1 second and doubles after each failed attempt:

```text
1s → 2s → 4s → 8s → 16s
```

Other HTTP errors are treated as non-retryable.

## Message sanitization

Before the generated message is returned:

* Backticks are removed.
* The first line is truncated to a maximum of 72 characters.

The action therefore provides a small amount of defensive cleanup even when a model does not fully follow the requested formatting.

## Complete example

This example obtains the changes between the previous and current commit and generates a commit message for them:

```yaml
name: Generate Commit Message

on:
  workflow_dispatch:

jobs:
  commit-message:
    runs-on: ubuntu-latest

    permissions:
      contents: read

    steps:
      - name: Check out repository
        uses: actions/checkout@v4
        with:
          fetch-depth: 2

      - name: Get diff
        id: diff
        shell: bash
        run: |
          {
            echo 'diff<<EOF'
            git diff HEAD^ HEAD
            echo 'EOF'
          } >> "$GITHUB_OUTPUT"

      - name: Generate commit message
        id: commit-message
        uses: amedee/ai-commit-message@v1
        env:
          OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
        with:
          diff_content: ${{ steps.diff.outputs.diff }}
          prompt_content: |
            Generate a concise commit message for the supplied Git diff.
            Describe what changed and why when this can be determined from the diff.
            Follow conventional commit conventions where appropriate.
            Return only the commit message.

      - name: Display commit message
        run: |
          echo "${{ steps.commit-message.outputs.message }}"
```

## OpenRouter

This action uses [OpenRouter](https://openrouter.ai/) to access AI models.

The action sends the following information to OpenRouter:

* The supplied `prompt_content`.
* The action's built-in commit-message instructions.
* The supplied Git diff.
* The selected model name.

The request also includes the GitHub repository as the HTTP `Referer` and identifies the application as `AI Commit Message`.

See the [OpenRouter documentation](https://openrouter.ai/docs) for information about the API and available models.

## Development

Clone the repository:

```bash
git clone https://github.com/amedee/ai-commit-message.git
cd ai-commit-message
```

Install dependencies:

```bash
npm ci
```

Build the action:

```bash
npm run build
```

This generates the bundled action in:

```text
dist/index.js
```

### Why is `dist/` committed?

This is a JavaScript GitHub Action, and `action.yml` points directly to the bundled file:

```yaml
runs:
  using: "node24"
  main: "dist/index.js"
```

Consumers of the action should not need to install npm dependencies or build the project themselves. Therefore, the generated `dist/` directory is committed to the repository.

After changing the source code or dependencies, rebuild the action:

```bash
npm ci
npm run build
```

Review the resulting changes to `dist/` and commit them together with the source changes.

### Versioning

The recommended way to consume the action is by major version:

```yaml
uses: amedee/ai-commit-message@v1
```

Specific versions can also be referenced:

```yaml
uses: amedee/ai-commit-message@v1.0.0
```

For maximum reproducibility, an exact commit SHA can be used.

### License

This project is licensed under the [ISC License](license).


[license]: https://opensource.org/license/isc
