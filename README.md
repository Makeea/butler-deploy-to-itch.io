# Deploying to Itch.io with GitHub Actions

Push your game to Itch.io (HTML, Windows, Mac, or Linux) straight from your GitHub repo. No em dashes, no nonsense.

## What You Need

* Game project in a GitHub repo
* Itch.io account: [https://itch.io/](https://itch.io/)
* Butler API key (your itch.io API key)
* Game project page already created on Itch.io: [https://itch.io/user/dashboard](https://itch.io/user/dashboard)

## 1. Get Your API Key

1. Log in at [https://itch.io/](https://itch.io/)
2. Go to [https://itch.io/user/settings/api-keys](https://itch.io/user/settings/api-keys)
3. Generate or copy your API key

Official docs: [https://itch.io/docs/butler/](https://itch.io/docs/butler/)

## 2. Add Your API Key to GitHub

1. In your repo, go to Settings > Secrets and variables > Actions
2. Click New repository secret
3. Name it: BUTLER\_API\_KEY
4. Paste your API key
5. Save

GitHub docs: [https://docs.github.com/en/actions/security-guides/encrypted-secrets](https://docs.github.com/en/actions/security-guides/encrypted-secrets)

## 3. Add the Workflow File

Create `.github/workflows/deploy-to-itchio.yml` in your repo:

```yaml
name: Deploy to Itch.io

on:
  push:
    branches:
      - main
  workflow_dispatch:

jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install Butler
        run: |
          curl -L -o butler.zip https://broth.itch.zone/butler/linux-amd64/LATEST/archive/default
          unzip butler.zip
          sudo mv butler /usr/local/bin/
          sudo chmod +x /usr/local/bin/butler

      # Deploy HTML build to Itch.io
      # Replace 'username/game-name' with your itch.io username and game name
      - name: Deploy to Itch.io (HTML)
        env:
          BUTLER_API_KEY: ${{ secrets.BUTLER_API_KEY }}
        run: |
          butler -V
          butler push www username/game-name:html

      # To deploy Windows build, uncomment below:
      # - name: Deploy to Itch.io (Windows)
      #   env:
      #     BUTLER_API_KEY: ${{ secrets.BUTLER_API_KEY }}
      #   run: |
      #     butler push build/windows username/game-name:windows

      # To deploy Mac build, uncomment below:
      # - name: Deploy to Itch.io (Mac)
      #   env:
      #     BUTLER_API_KEY: ${{ secrets.BUTLER_API_KEY }}
      #   run: |
      #     butler push build/macos username/game-name:mac

      # To deploy Linux build, uncomment below:
      # - name: Deploy to Itch.io (Linux)
      #   env:
      #     BUTLER_API_KEY: ${{ secrets.BUTLER_API_KEY }}
      #   run: |
      #     butler push build/linux username/game-name:linux

      # To deploy all at once, you can combine the commands in a single run:
      # - name: Deploy all builds to Itch.io (HTML, Windows, Mac, Linux)
      #   env:
      #     BUTLER_API_KEY: ${{ secrets.BUTLER_API_KEY }}
      #   run: |
      #     butler -V
      #     butler push www username/game-name:html
      #     butler push build/windows username/game-name:windows
      #     butler push build/macos username/game-name:mac
      #     butler push build/linux username/game-name:linux
```

## Butler Command Breakdown

```bash
butler push build/macos username/game-name:mac
```

* `butler push` runs Butler to upload your build to Itch.io.
* `build/macos` is your build output folder. Use whatever folder your build process creates: `build/macos` for Mac, `build/windows` for Windows, `dist` or `www` for HTML, etc.
* `username/game-name` is your Itch.io username and your game’s URL-friendly name (the "slug").
* `:mac` is the channel on Itch.io. Use `:mac` for Mac, `:windows` for Windows, `:linux` for Linux, or `:html` for web builds.

Universal example:

```bash
butler push www username/game-name:html      # HTML build from the www folder
butler push build/windows username/game-name:windows
butler push build/macos username/game-name:mac
butler push build/linux username/game-name:linux
```

Real-world example:

```bash
butler push www makeea/chrono-courier:html
butler push build/windows makeea/chrono-courier:windows
butler push build/macos makeea/chrono-courier:mac
butler push build/linux makeea/chrono-courier:linux
```

* The first part (www, build/macos, etc.) is your local build output folder.
* The second part is always your username, your game’s name, and the platform channel.
* Channels control which platform tab users will see on your Itch page.
* Add or remove build steps for Windows, Mac, Linux as needed

## 4. How To Use

* Push to main to deploy automatically
* Manual trigger: Go to Actions, find this workflow, click Run workflow
* Logs and errors appear in your repo’s Actions tab

## 5. Troubleshooting

* No deploy? Double-check your API key, spelling, and that the secret exists
* Butler errors? Make sure the www folder (or other build output) exists before upload
* Build not showing up? Wait a few minutes. Itch.io can process builds for a short time
* More: [https://itch.io/docs/butler/](https://itch.io/docs/butler/)

## Useful Links

* Butler CLI Docs: [https://itch.io/docs/butler/](https://itch.io/docs/butler/)
* Create a Game Project: [https://itch.io/game/new](https://itch.io/game/new)
* GitHub Actions Quickstart: [https://docs.github.com/en/actions/quickstart](https://docs.github.com/en/actions/quickstart)
* Itch.io Forums: [https://itch.io/board](https://itch.io/board)

## FAQ

* Q: Can I use this with private repos?
  A: Yes, but your Actions runner must have access to your secret
* Q: Can I trigger deploys only for tags or releases?
  A: Change the on: section to suit your needs

If you’re still stuck, check the logs or read the docs.
