# GitHub Pages Hosting

## Recommended approach

Because this repo now has a `docs/` folder, the easiest hosting option is **GitHub Pages from the `main` branch and `/docs` folder**.

## What is already prepared

This repo includes:

- `docs/index.html` for Docsify
- `docs/README.md` as the landing page
- `docs/_sidebar.md` for navigation
- `docs/.nojekyll` for GitHub Pages compatibility

## Steps to host on GitHub Pages

### 1. Push the repo to GitHub

If this repository is not yet on GitHub:

```bash
git init
git add .
git commit -m "Add Docsify AWS Lambda upskilling docs"
git branch -M main
git remote add origin <your-github-repo-url>
git push -u origin main
```

If the repo already exists remotely, just commit and push your changes.

### 2. Open repository settings

In GitHub:

- Open the repository
- Go to **Settings**
- Open **Pages** in the left sidebar

### 3. Configure the source

Under **Build and deployment**:

- Source: **Deploy from a branch**
- Branch: **main**
- Folder: **/docs**
- Save

### 4. Wait for deployment

GitHub Pages usually takes a minute or two.

Your site URL will usually look like:

```text
https://<your-github-username>.github.io/<repo-name>/
```

## Important Docsify note

Docsify runs fully in the browser, so GitHub Pages works well for it.
You do not need a separate build step for this setup.

## If the site shows a blank page

Check these things:

- `docs/index.html` exists
- `docs/.nojekyll` exists
- GitHub Pages is pointing to the `main` branch and `/docs`
- The repository has been pushed successfully
- Wait a minute and refresh

## Optional local preview

You can preview the docs locally with a simple static server.

If you have Python installed:

```bash
python3 -m http.server 3000
```

Then open:

```text
http://localhost:3000/docs/
```

## Final result

Once GitHub Pages is enabled, your beginner-friendly AWS Lambda docs site will be publicly accessible from your GitHub Pages URL.
