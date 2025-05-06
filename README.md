# Locawise GitHub Action (`locawise-action`)

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT) 

**Automate your localization workflow with AI!** This GitHub Action uses the `locawise` Python package to automatically detect changes in your localization files, translate them using powerful AI models (OpenAI or Google Vertex AI), and create a Pull Request with the updates.

**Go from supporting 2-3 languages to virtually all languages for the price of a coffee! ☕️**

## Quick Start
Setting up Locawise is incredibly simple:

1. Add an i18n.yaml configuration file to your repository root
2. Create a GitHub Actions workflow file (.github/workflows/localize.yml)
3. Locawise will automatically handle translations whenever you push changes

No complex setup, no maintenance headaches - just effortless localization powered by AI.


## Table of Contents

* [Why Use Locawise Action?](#why-use-locawise-action)
* [How it Works](#how-it-works)
* [Prerequisites](#prerequisites)
* [Configuration (`i18n.yaml`)](#configuration-i18nyaml)
* [Usage Examples](#usage-examples)
  * [1. Spring Boot (.properties) with Vertex AI](#1-spring-boot-properties-with-vertex-ai)
  * [2. React (.json) with Vertex AI](#2-react-json-with-vertex-ai)
  * [3. React (.json) with OpenAI (GPT)](#3-react-json-with-openai-gpt)
* [Action Inputs](#action-inputs)
* [Required Permissions](#required-permissions)
* [Under the Hood: `locawise`](#under-the-hood-locawise)
* [Supported Features](#supported-features)
* [Cost Considerations](#cost-considerations)
* [Contributing](#contributing)
* [License](#license)

## Why Use Locawise Action?

* **Fully Automated:** Set it up once, and let AI handle localization updates on every push.
* **AI-Powered Translations:** Leverage state-of-the-art LLMs (OpenAI GPT models, Google Vertex AI Gemini models) for high-quality, context-aware translations.
* **Context & Glossary Aware:** Define your application's context, tone, and glossary directly in a config file (`i18n.yaml`) for accurate results.
* **Seamless Integration:** Fits directly into your existing GitHub workflow. Detects changes, translates, and creates a PR automatically.
* **Cost-Effective:** Pay only for the LLM usage – potentially near-zero cost with efficient models like Gemini.
* **Efficient:** Uses a lock file (`i18n.lock`) to only translate *new* or *changed* keys, saving time and cost.
* **Simple Setup:** Requires minimal configuration in your workflow file and a straightforward YAML config for `locawise`.

## How it Works

1. **Trigger:** Runs on specific events in your repository (e.g., `push` to the main branch).
2. **Checkout:** Checks out your repository code.
3. **Setup:** Installs Python and the `locawise` package.
4. **Authenticate (Optional):** Authenticates with Google Cloud if using Vertex AI.
5. **Run Locawise:** Executes the `locawise` command-line tool, pointing it to your `i18n.yaml` configuration file.
   * `locawise` parses the config and compares your source language file with target language files using `i18n.lock`.
   * Identifies new or modified keys.
   * Sends these keys (with context, glossary, and tone) to the configured LLM for translation.
   * Updates the target language files.
   * Updates the `i18n.lock` file.
6. **Check for Changes:** Determines if `locawise` modified any localization files.
7. **Create Pull Request:** If changes were detected, it automatically creates a Pull Request with the updated localization files using [peter-evans/create-pull-request](https://github.com/peter-evans/create-pull-request).

## Prerequisites

1. **GitHub Repository:** Your project hosted on GitHub.
2. **Localization Files:** Your application's strings stored in supported formats (currently `.json` or `.properties`).
3. **`locawise` Configuration File:** An `i18n.yaml` file in your repository (see [Configuration](#configuration-i18nyaml)).
4. **LLM API Credentials:**
   * For **OpenAI:** An API key (`OPENAI_API_KEY`).
   * For **Vertex AI:** A Google Cloud Service Account Key JSON file with permissions for Vertex AI.
5. **GitHub Secrets:** Store your API credentials securely as GitHub Secrets (e.g., `OPENAI_API_KEY`, `VERTEX_AI_SERVICE_ACCOUNT_KEY_BASE64`). **Never commit secrets directly to your repository!**
6. **Workflow Permissions:** Your GitHub workflow must have `contents: write` and `pull-requests: write` permissions.
7. **Enable GitHub Actions to Create PRs:** Some repositories may need to explicitly enable GitHub Actions to create pull requests. You can do this in your repository's Settings → Actions → General → Workflow permissions, and ensure "Allow GitHub Actions to create and approve pull requests" is checked.

## Configuration (`i18n.yaml`)

This action relies on the `locawise` Python package, which requires a configuration file (e.g., `i18n.yaml`) **in your repository root**, following the structure below.

**Example `i18n.yaml`:**

```yaml
# Localization Configuration
version: v1.0 # Config schema version (required)

localization-root-path: ./path/to/your/localization/files # Relative path from repo root (required)
file-name-pattern: "{language}.json" # Use {language} placeholder (required). E.g., en.json, tr.json
source-lang-code: en # Source language code (required)
target-lang-codes: # List of target language codes (required)
  - tr
  - it
  - es
  - fr
  # Add as many languages as you need!

# --- AI Configuration ---
llm-model: gemini-1.5-flash # Optional: Specify the exact LLM model (e.g., 'gpt-4o', 'gemini-1.5-pro')
llm-location: us-central1 # Optional: Specify the LLM location/region if applicable (e.g., for Vertex AI)

# --- Context, Glossary & Tone (Crucial for Quality!) ---
context: | # Optional: Detailed instructions for the AI about your app, audience, etc.
  You are translating for a fintech app called Verolut.
  Verolut offers business accounts for SMEs and freelancers.
  Target Audience: Small business owners, freelancers in USA.

glossary: # Optional: Define key terms and their required translations
  merchant: Business using Verolut (company, sole trader).
  member: Individual user linked to a merchant account.
  ledger: Bank account.
  # Add your specific terminology

tone: Professional but friendly # Optional: Specify the desired tone for translations
```

### Key Fields:

| Field | Requirement | Default Value | Description |
|-------|-------------|---------------|-------------|
| **version** | Required | None | The version of the configuration schema being used. |
| **source-lang-code** | Required | None | The language code of your primary language file (e.g., en). |
| **file-name-pattern** | Required | None | How your language files are named. {language} is replaced by the language code (e.g., en, tr). |
| **target-lang-codes** | Required | None | A list of language codes you want to translate into. |
| **localization-root-path** | Optional | Empty string ("") | Path to the directory containing your language files (relative to repo root). |
| **context** | Optional | Empty string ("") | Detailed instructions for the AI about your application, tone, style, and target audience. The more detail, the better the translation! |
| **glossary** | Optional | Empty dictionary | Define specific terms and their required translations to ensure consistency. Keys are the source term, values are the target term (though the AI uses this for context across all target languages). |
| **tone** | Optional | Empty string ("") | A description of the desired tone (e.g., "Formal", "Playful", "Professional but friendly"). |
| **llm-model** | Optional | None | The specific model name from the provider (e.g., gpt-4o, gemini-1.5-flash). If omitted, locawise will use a default. |
| **llm-location** | Optional | None | The geographic location or region for the LLM API endpoint, if applicable (e.g., for Vertex AI). |

## Usage Examples

Here are a few examples demonstrating how to configure the workflow and i18n.yaml for common scenarios.

### 1. Spring Boot (.properties) with Vertex AI

This example assumes your Spring Boot project stores localization files in src/main/resources/i18n with names like messages_en.properties, messages_fr.properties, etc. We'll use Google Vertex AI for translation.

**Workflow Setup (.github/workflows/localize.yml):**

```yaml
name: Automatic Localization (Spring Boot)

on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write

jobs:
  localize:
    runs-on: ubuntu-latest
    name: Run Locawise AI Localization (Vertex AI)
    steps:
      - name: Run Locawise Action
        uses: aemresafak/locawise-action@v1 # <-- Use the latest version tag
        with:
          # Provide ONLY the Vertex AI key secret for Vertex AI models
          vertex-ai-service-account-key-base64: ${{ secrets.VERTEX_AI_SERVICE_ACCOUNT_KEY_BASE64 }}
```

**Configuration (i18n.yaml):**

```yaml
version: v1.0
source-lang-code: en
target-lang-codes:
  - fr
  - de
  - es
localization-root-path: ./src/main/resources/i18n # Path to your i18n directory
file-name-pattern: "messages_{language}.properties" # Matches files like messages_en.properties

# --- AI Configuration (Vertex AI) ---
llm-model: gemini-1.5-flash # Or another Gemini model
llm-location: us-central1 # Specify your Vertex AI region

# --- Context, Glossary & Tone ---
context: |
  Translating for a Spring Boot backend application providing financial services.
  Maintain a formal and secure tone.
glossary:
  transaction: financial operation
  balance: account amount
tone: Formal
```

### 2. React (.json) with Vertex AI

This example assumes your React project stores localization files in src/locales with names like en.json, fr.json, etc. We'll use Google Vertex AI for translation.

**Workflow Setup (.github/workflows/localize.yml):**

```yaml
name: Automatic Localization (React)

on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write

jobs:
  localize:
    runs-on: ubuntu-latest
    name: Run Locawise AI Localization (Vertex AI)
    steps:
      - name: Run Locawise Action
        uses: aemresafak/locawise-action@v1 # <-- Use the latest version tag
        with:
          # Provide ONLY the Vertex AI key secret for Vertex AI models
          vertex-ai-service-account-key-base64: ${{ secrets.VERTEX_AI_SERVICE_ACCOUNT_KEY_BASE64 }}
```

**Configuration (i18n.yaml):**

```yaml
version: v1.0
source-lang-code: en
target-lang-codes:
  - fr
  - de
  - es
localization-root-path: ./src/locales # Path to your locales directory
file-name-pattern: "{language}.json" # Matches files like en.json

# --- AI Configuration (Vertex AI) ---
llm-model: gemini-1.5-flash
llm-location: us-central1

# --- Context, Glossary & Tone ---
context: |
  Translating for a React frontend application for an e-commerce platform.
  Target audience: General consumers.
glossary:
  cart: shopping basket
  checkout: payment process
tone: Friendly and inviting
```

### 3. React (.json) with OpenAI (GPT)

This example uses the same React project structure (src/locales/{language}.json) but utilizes OpenAI's GPT models for translation.

**Workflow Setup (.github/workflows/localize.yml):**

```yaml
name: Automatic Localization (React)

on:
  push:
    branches:
      - main

permissions:
  contents: write
  pull-requests: write

jobs:
  localize:
    runs-on: ubuntu-latest
    name: Run Locawise AI Localization (OpenAI)
    steps:
      - name: Run Locawise Action
        uses: aemresafak/locawise-action@v1 # <-- Use the latest version tag
        with:
          # Provide ONLY the OpenAI key secret for OpenAI models
          openai-api-key: ${{ secrets.OPENAI_API_KEY }}
```

**Configuration (i18n.yaml):**

```yaml
version: v1.0
source-lang-code: en
target-lang-codes:
  - fr
  - de
  - es
localization-root-path: ./src/locales # Path to your locales directory
file-name-pattern: "{language}.json" # Matches files like en.json

# --- AI Configuration (OpenAI) ---
llm-model: gpt-4o # Or another OpenAI model (e.g., gpt-4-turbo)

# --- Context, Glossary & Tone ---
context: |
  Translating for a React frontend application for an e-commerce platform.
  Target audience: General consumers.
glossary:
  cart: shopping basket
  checkout: payment process
tone: Friendly and inviting
```

## Action Inputs

| Input | Description | Required | Default | Secret? |
|-------|-------------|----------|---------|---------|
| openai-api-key | Your OpenAI API key. Provide this if you intend to use OpenAI models. | No | '' | Yes |
| vertex-ai-service-account-key-base64 | Your Vertex AI Service Account Key JSON, base64 encoded. Provide this if using Vertex AI. | No | '' | Yes |

**Note on Vertex AI Key**: You need to base64 encode your service account JSON key before adding it as a secret. You can do this on Linux/macOS with: `base64 -w 0 service-account-key.json` (replace service-account-key.json with your file name). On Windows, you can use `certutil -encode -f service-account-key.json encoded-key.txt` and then copy the contents of encoded-key.txt.

## Required Permissions

The action requires the following permissions set at the top level of your workflow file:

- **contents: write**: To commit the updated localization files and the i18n.lock file back to the repository branch created for the PR.
- **pull-requests: write**: To create the Pull Request with the localization changes.

```yaml
permissions:
  contents: write
  pull-requests: write
```

## Under the Hood: `locawise`

This action is a wrapper around the `locawise` Python package. `locawise` handles the core logic:
- Parsing the `i18n.yaml` configuration.
- Reading source and target language files.
- Using `i18n.lock` to track translated keys and avoid re-translating unchanged content.
- Interacting with the configured LLM API (OpenAI or Vertex AI).
- Implementing resilience features like exponential backoff for API rate limits.

For more details on the `locawise` package itself, please refer to its repository.

## Supported Features

- **File Formats**: `.json` (key-value), `.properties`
- **LLM Providers**: OpenAI, Google Vertex AI
- **Automation**: Automatic change detection and Pull Request creation.

Future plans may include support for more file formats (e.g., YAML, XML, ARB).

## Cost Considerations
Costs depend on the LLM provider you use:

**OpenAI:** Charged per token according to OpenAI pricing
**VertexAI (Gemini):** Extremely cost-effective, nearly zero for most projects

## Contributing

Contributions are welcome! Please feel free to submit Issues or Pull Requests.

## License

This project is licensed under the MIT License.

---
Made with ❤️ by aemresafak
