# Locawise GitHub Action (locawise-action)

Stop wrestling with manual translations! Automate your app localization with the power of AI! 🚀

`locawise-action` seamlessly integrates into your GitHub workflow, using the intelligent `locawise` Python tool to:

*   Detect new or changed text in your app's primary language file.
*   Translate it using cutting-edge AI models (OpenAI or Google Vertex AI).
*   Create a Pull Request automatically with all your updated translations.

Go from supporting 2-3 languages to virtually ALL languages, for less than your daily coffee! ☕️

## Why You'll Love Locawise Action

*   **🌍 Effortless Global Reach:** Automate localization and connect with users worldwide.
*   **🧠 Smart AI Translations:** Leverage models like GPT-4o or Gemini for high-quality, context-aware translations. You define the context, glossary, and tone!
*   **⚙️ Seamless Workflow:** Fits directly into your GitHub CI/CD. Push your code, and get PRs with fresh translations.
*   **💰 Incredibly Cost-Effective:** Only pay for the LLM usage. With efficient models like Gemini, costs can be astonishingly low!
*   **⏱️ Lightning Fast Setup:** Get up and running in minutes with a simple YAML configuration and a few lines in your workflow file.
*   **✅ Efficient & Respectful:** `locawise` intelligently translates only new or changed content and respects any manual modifications you've made to the target language files.

## Quick Start - Go Global in 3 Easy Steps!

1.  **Create your `i18n.yaml` configuration file** in your project's root directory. This file tells `locawise` how to handle your translations:

    ```yaml
    # Localization Configuration for Locawise
    # This file guides the locawise tool in translating your application.
    # Ensure this file is placed in the root of your repository.

    version: v1.0 # Required: Specifies the schema version of this configuration file.
                  # Adherence to this version ensures compatibility with locawise.

    localization-root-path: ./src/locales # Required: Relative path from your repository's root
                                         # to the directory containing your language files.
                                         # Example: "./src/locales", "./config/i18n", "./app/languages"

    file-name-pattern: "{language}.json" # Required: Defines the naming convention for your language files.
                                       # The "{language}" placeholder will be dynamically replaced
                                       # by the language code (e.g., "en.json", "fr.json").

    source-lang-code: en # Required: The language code of your primary (source) language file
                         # (e.g., "en" for English, "es" for Spanish). This is the language
                         # from which all translations will be generated.

    target-lang-codes: # Required: A list of language codes for the languages you want
                       # to translate your application into.
      - fr # Example: French
      - es # Example: Spanish
      - de # Example: German
      # - ja # Optional: Add as many target languages as you need, like Japanese here.

    # --- Translation Context & Quality (Optional, but HIGHLY RECOMMENDED for superior translations!) ---

    # context: | # Optional: Provide detailed instructions and background for the AI.
    #            # Describe your application, its purpose, target audience, brand voice,
    #            # and any specific stylistic requirements. The more comprehensive the context,
    #            # the more accurate and nuanced the translations will be.
    #   You are translating for "ConnectSphere", a professional networking platform.
    #   Our target audience is early-career professionals and university students.
    #   The tone should be encouraging, professional, yet approachable.
    #   Avoid overly casual slang but maintain a friendly voice.
    #   Key features include "Mentorship Matching" and "Skill Endorsements".

    # glossary: # Optional: Define key terms, brand names, specific jargon, or phrases
    #           # and their required translations (or specify they should not be translated).
    #           # This ensures consistency and accuracy across all target languages.
    #           # Format: source_term: preferred_translation_or_directive
    #   ConnectSphere: This is the application name. # Ensures brand name is not translated
    #   Skill Endorsement: Competency Validation # Example for a specific term
    #   Lingos: The total languages a person can speak 

    # tone: Professional yet approachable # Optional: Briefly describe the desired overall tone for the
    #                                     # This acts as a quick guide for the AI, complementing the 'context' field.

    # --- AI Configuration (Optional - sensible defaults are used if not specified) ---

    # llm-model: gemini-1.5-flash # Optional: Specify the exact LLM model you wish to use.
    #                             # Examples: 'gpt-4o', 'gpt-4-turbo' (for OpenAI)
    #                             #           'gemini-1.5-pro', 'gemini-1.0-pro' (for Vertex AI)
    #                             # If omitted, locawise uses a default model from the detected provider.

    # llm-location: us-central1 # Optional: For some LLM providers (like Google Vertex AI),
    #                           # you can specify the geographic location or region for the API endpoint.
    #                           # Consult your LLM provider's documentation for available regions.

    ```

2.  **Set up your GitHub Actions workflow** (e.g., `.github/workflows/localize.yml`):

    ```yaml
    name: AI-Powered Localization

    on:
      push:
        branches: [ main ] # Or your primary development branch

    permissions:
      contents: write      # Required to commit changes to a new branch
      pull-requests: write # Required to create the pull request

    jobs:
      localize:
        runs-on: ubuntu-latest
        name: Run Locawise AI Localization
        steps:
          - name: Checkout Repository
            uses: actions/checkout@v4

          - name: Run Locawise Action
            uses: aemresafak/locawise-action@v1 # Make sure to use the latest version
            with:
              # --- Choose ONE of the following based on your LLM Provider ---

              # For OpenAI (e.g., GPT-4o, GPT-3.5-turbo):
              openai-api-key: ${{ secrets.OPENAI_API_KEY }}

              # OR

              # For Google Vertex AI (e.g., Gemini models):
              # vertex-ai-service-account-key-base64: ${{ secrets.VERTEX_AI_SERVICE_ACCOUNT_KEY_BASE64 }}
    ```

3.**Add your LLM API keys** to your repository's GitHub Secrets. Go to Settings > Secrets and variables > Actions > New repository secret.

- For OPENAI_API_KEY, paste your key directly.

- For VERTEX_AI_SERVICE_ACCOUNT_KEY_BASE64, you must first base64 encode your JSON service account key file.

    - On Linux/macOS: base64 -w 0 your-service-account-key.json

    - On Windows (PowerShell): [Convert]::ToBase64String([IO.File]::ReadAllBytes("your-service-account-key.json"))

    - Then, paste the resulting base64 string as the secret.

That's it! Now, whenever you push changes to your specified branch, `locawise-action` will automatically handle the translation updates.

## How It Works - The Magic Unveiled

1.  **Push Code:** You push your latest code changes to the configured branch.
2.  **Detect Changes:** `locawise` intelligently compares your source language file against the target files, using an `i18n.lock` file to track already translated content. It identifies only new or modified strings.
3.  **AI Translation:** It sends these specific strings, along with the rich context, glossary, and tone you defined in `i18n.yaml`, to your chosen AI model (OpenAI or Vertex AI).
4.  **Create Pull Request:** The action commits the newly translated language files and the updated `i18n.lock` file to a new branch, then creates a Pull Request for you to easily review and merge.

## Supported Features

*   **File Formats:** `.json` (key-value, including nested structures), `.properties` (More formats like YAML, ARB are planned!)
*   **AI Models:** OpenAI (GPT series), Google Vertex AI (Gemini series). More providers coming soon!
*   **Context-Awareness:** Deeply customize translations using context, glossary, and tone instructions.
*   **Efficiency:** Smart change detection via `i18n.lock` means you only pay for translating what's new.
*   **Resilience:** Built-in retries with exponential backoff to handle temporary LLM API limits.

## Dive Deeper 📚

*   ➡️ [Full Documentation & Advanced Configuration](Link to your future detailed documentation site)
*   ➡️ [Usage Examples (Spring Boot, React, etc.)](Link to usage examples)
*   ➡️ [Learn about the locawise Python package](Link to the locawise Python package)

## Contributing

Your ideas and contributions can make Locawise even better! Please feel free to submit Issues or Pull Requests.

## License

MIT License © aemresafak
