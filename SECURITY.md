# Security Policy

## Reporting a Vulnerability

Please **do not report security vulnerabilities through public GitHub issues**.

If you believe you have found a security vulnerability in this project, please report it privately through [GitHub's private vulnerability reporting](https://github.com/amedee/ai-commit-message/security/advisories/new).

Please include:

* A description of the vulnerability
* The affected version or commit
* Steps to reproduce the issue
* The potential impact
* Any suggested mitigation, if available

You will receive a response as soon as reasonably possible.

## API Keys and Secrets

This action uses the `OPENROUTER_API_KEY` environment variable to authenticate with OpenRouter.

API keys should always be stored as GitHub Actions secrets:

```yaml
env:
  OPENROUTER_API_KEY: ${{ secrets.OPENROUTER_API_KEY }}
```

Never commit API keys or other credentials to the repository.

## Data sent to OpenRouter

The action sends the following information to the OpenRouter API:

* The contents of `diff_content`
* The contents of `prompt_content`
* The action's built-in instructions
* The selected OpenRouter model identifier

Do not provide sensitive information through these inputs unless you are comfortable with that information being processed by the selected OpenRouter model and its applicable providers.

Consult the [OpenRouter Privacy Policy](https://openrouter.ai/privacy) and the documentation for the model you are using for information about data handling.

## Supported Versions

Security fixes are applied to actively maintained versions of the action.

Consumers are encouraged to use the latest release within their chosen major version.

For example:

```yaml
uses: amedee/ai-commit-message@v1
```

Pinning to a specific commit SHA provides additional protection against unexpected changes:

```yaml
uses: amedee/ai-commit-message@<commit-sha>
```

## Dependencies

This project uses npm dependencies. Dependency updates are monitored through Dependabot.

The generated `dist/` bundle is verified by CI to ensure it matches the committed source.
