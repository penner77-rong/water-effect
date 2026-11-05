# Publish this version on GitHub

Prepared November 5, 2026. The prepared project folder is `hello-world-water-effect`. Its `index.html` is the synchronized current prototype. The workspace also has a dated HTML checkpoint.

## Upload with the browser

1. Sign in to GitHub and open [New repository](https://github.com/new).
2. Name it `hello-world-water-effect`. Select Public if you want a free public GitHub Pages demo. Leave README, .gitignore, and license initialization empty since this folder already contains the first two.
3. Click Create repository.
4. On the empty repository screen choose “uploading an existing file.” In an existing repository, use Add file → Upload files.
5. Open the prepared folder in Finder. Drag `index.html`, `README.md`, `WATER-EFFECT-NOTES.md`, `WATER-EFFECT-PRD.md`, `GITHUB-PUBLISHING.md`, and the `audio` folder into the upload area. Upload the contents so index.html is at the repository root. For the hidden .gitignore file, press Command+Shift+. in Finder and include it. Leave .git and .DS_Store out of the upload.
6. Check that both audio files appear inside `audio/`.
7. Enter the commit message `Document and publish November 5 water prototype`, then commit the upload to main.
8. Open index.html in the repository to verify the uploaded file exists.

GitHub's browser upload supports up to 100 files per batch and 25 MiB per file. All files in this prepared project fit those limits.

## Publish the interactive website

1. In the repository open Settings → Pages.
2. Under Build and deployment select Deploy from a branch.
3. Choose main and /(root), then Save.
4. Wait for the Pages deployment to finish. Check the Actions tab if deployment fails.
5. Return to Settings → Pages and use the displayed site link. For this repository name, it normally follows `https://YOUR-USERNAME.github.io/hello-world-water-effect/`.
6. Test a click, slow and fast movement, Sound, and the control panel. An HTTPS site can request supported local font access when you click Browse installed fonts.

## Update later

Synchronize the working prototype into this folder's index.html. Upload the changed files using Add file → Upload files and commit a descriptive message. Pages redeploys from main. Browser uploads create GitHub history, while the pre-existing local Git folder stays unsynchronized. If you later adopt Git or GitHub Desktop, clone the published repository into a new folder and use that clone for future commits.

Panel edits are temporary. To publish a customized default appearance, save those defaults into the source HTML first.

## Baseline identity

SHA-256 for the November 5 index.html is `f6b832c73ee801cc35752b7d7c8a4ea2d2b27a20c218274740ff4fbf8f468ce6`.

## GitHub instructions

- [Create a repository](https://docs.github.com/en/repositories/creating-and-managing-repositories/creating-a-new-repository)
- [Upload files](https://docs.github.com/en/repositories/working-with-files/managing-files/adding-a-file-to-a-repository)
- [Configure GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site)
