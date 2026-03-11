# AWS Lambda Upskilling

Beginner-friendly AWS Lambda learning documentation built with Docsify and designed for GitHub Pages hosting.

## Documentation site

The docs site lives in the `docs/` folder and includes:

- A beginner-friendly Lambda learning path
- Core concepts explained simply
- First Lambda walkthroughs
- Integration examples for API Gateway, S3, EventBridge, and SQS
- Best practices and a mini project
- GitHub Pages hosting instructions

## Project structure

```text
.
├── README.md
└── docs/
    ├── .nojekyll
    ├── _coverpage.md
    ├── _sidebar.md
    ├── README.md
    ├── index.html
    ├── foundations.md
    ├── core-concepts.md
    ├── first-lambda.md
    ├── integrations.md
    ├── best-practices.md
    ├── mini-project.md
    └── github-pages.md
```

## Run locally

You can preview the site locally with a simple static file server.

Using Python:

```bash
python3 -m http.server 3000
```

Then open:

```text
http://localhost:3000/docs/
```

## Host with GitHub Pages

### 1. Push your code to GitHub

```bash
git add .
git commit -m "Add Docsify AWS Lambda upskilling docs"
git push
```

### 2. Configure GitHub Pages

In your GitHub repository:

- Open `Settings`
- Open `Pages`
- Under `Build and deployment`, select `Deploy from a branch`
- Choose branch `main`
- Choose folder `/docs`
- Save

### 3. Access your site

Your site URL will usually be:

```text
https://<your-github-username>.github.io/<repo-name>/
```

## Notes

- Docsify works well with GitHub Pages because it does not require a build step for this setup.
- The `docs/.nojekyll` file is included for compatibility.
- The main learning content starts at `docs/README.md`.