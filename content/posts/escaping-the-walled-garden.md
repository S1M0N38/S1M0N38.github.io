+++
title = 'Escaping the Walled Garden'
date = 2024-01-27T14:00:00+01:00
draft = false
+++

GitHub Copilot is a cloud-based AI tool for code auto-completion developed by GitHub and OpenAI, released in 2021. Nowadays, Microsoft (MS) [owns GitHub](https://www.theverge.com/2018/10/26/17954714/microsoft-github-deal-acquisition-complete) (since October 2018) and [49% of OpenAI](https://www.theverge.com/2023/1/23/23567448/microsoft-openai-partnership-extension-ai) (since January 2023). Meanwhile, MS developed [VS Code](https://survey.stackoverflow.co/2023/#section-most-popular-technologies-integrated-development-environment), the most used code editor (not the most admired, though). So, MS plays its cards well enough to become the undisputed leader in the developers' ecosystem, owning the entire supply chain:

- **raw material**: Unrestricted access to GitHub public repositories.

- **manufacturing**: OpenAI know-how and technology.

- **addictive products**: Copilot and GPT-4.

- **distribution**: VS Code marketplace.

Even though this has proved to be a well-oiled developers' experience, I'd like to be able to swap some of its parts to my liking.

## Copilot proxy

Although some ways to use Copilot auto-completion are officially supported in [various editors](https://docs.github.com/en/copilot/using-github-copilot/getting-started-with-github-copilot), only VS Code extensions make use of its [full potential](https://code.visualstudio.com/docs/editor/github-copilot).

One powerful feature that Copilot exposes through the [GitHub Copilot Chat](https://marketplace.visualstudio.com/items?itemName=GitHub.copilot-chat) extension is the ability to query GPT-4 directly from the editor, asking code-related questions. I want that in other places of the development workflow.

> A simple proxy should be enough to redirect requests to the Copilot API: [S1M0N38/copilot](https://github.com/S1M0N38/copilot), use it at your own risk.

## Applications

Now that a breach in the walled garden has been opened, GPT-4 can be used in other applications that make requests to the following OpenAI endpoints:

- `[/v1]/chat/completions`
- `[/v1]/embeddings`

### Neovim

Plenty of AI plugins make use of OpenAI endpoints. Just look for the option that lets you choose the AI model and the base URL, for example:

```lua
-- using layz.nvim ...
{
  { -- my own plugin :)
    "S1M0N38/dante.nvim",
    opts = {base_url = "http://localhost:4141"},
  },
  { -- Pull Request #59 (still pending)
    "Bryley/NeoAI.nvim",
    opts = {open_ai = {base_url = "http://localhost:4141"}},
  },
  { -- export OPENAI_API_HOST="http://localhost:4141"
    "jackMort/ChatGPT.nvim",
    opts = {},
  },
  -- Other AI plugins ...
}
```

You get the idea. Almost every plugin that uses OpenAI endpoints lets you choose the base URL in some way.

### Jupyter Lab

Assuming that you have installed Jupyter Lab with [pipx](https://pipx.pypa.io/stable/) as suggested [here](https://samedwardes.com/2022/10/23/best-jupyter-lab-install/) using the following command:

```sh
pipx install jupyterlab --include-deps
```

You can add the AI chat extension using these commands:

```sh
pipx inject jupyterlab 'jupyter_ai'
pipx inject jupyterlab 'openai'
```

Configure the extension with the following settings:

- **Language Model**
  - *Language model*: `OpenAI::gpt-4`
  - *Base URL API*: `http://localhost:4141`
- **Embedding Model**:
  - *Embedding model*: `OpenAI::text-embedding-ada-002`
  - *Base URL API*: `http://localhost:4141`

At the time of writing, the Embedding model does not support the Base URL API. Keep an eye on [this issue](https://github.com/jupyterlab/jupyter-ai/issues/587) for updates.

### LibreChat

The cherry on top is [LibreChat](https://docs.librechat.ai/), an open-source clone of the ChatGPT interface. Set these variables in the `.env` file

```sh
OPENAI_API_KEY=sk-1234
OPENAI_MODELS=gpt-4,gpt-3.5-turbo
OPENAI_REVERSE_PROXY=http://localhost:4141/v1/chat/completions
```

Maybe there is a better way for configuring LibreChat, but this way works.
