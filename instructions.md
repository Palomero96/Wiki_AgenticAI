# Guide: Create a Site with MkDocs and Deploy to GitHub Pages

Welcome to your new documentation! MkDocs is a fast, simple static site generator that's geared towards building project documentation. Documentation source files are written in Markdown, and configured with a single YAML configuration file.

In this guide, you will learn how to initialize this project and publish it for free using GitHub Pages.

## Prerequisites

Before you begin, make sure you have the following installed on your system:
* Python (and pip to install packages).
* Git for version control.
* A GitHub account.

## Step 1: Install MkDocs

Open your terminal and run the following command to install MkDocs. We recommend also installing the "Material" theme, which is the most popular and modern:

```bash
pip install mkdocs mkdocs-material
```

## Step 2: Create a New Project

To create a new project, run the new command followed by the name you want to give your project folder (for example, my-documentation):

```bash
mkdocs new my-documentation
cd my-documentation
```


This will generate the following file structure:

```text
my-documentation/
├─ mkdocs.yml    # The main configuration file
└─ docs/
   └─ index.md   # The main page (You can paste this exact text here!)
```

## Step 3: Configure Your Site

Open the mkdocs.yml file in your favorite code editor and modify it to look like this:

```yaml
site_name: My Awesome Documentation
site_url: https://<YOUR-USERNAME>.github.io/<YOUR-REPOSITORY>/

theme:
  name: material

nav:
  - Home: index.md
  - About: about.md
```

(Be sure to replace <YOUR-USERNAME> and <YOUR-REPOSITORY> with your actual GitHub details).

## Step 4: Local Preview

To see how your site looks as you write, use the built-in development server:

```bash
mkdocs serve
```


Open your browser and go to http://127.0.0.1:8000. Any changes you make to your Markdown files will automatically update in the browser.

## Step 5: Push the Project to GitHub

Go to GitHub and create a new empty repository (e.g., my-documentation).

In your terminal, initialize Git and push your code:

```bash
git init
git add .
git commit -m "Initialize MkDocs project"
git branch -M main
git remote add origin https://github.com/<YOUR-USERNAME>/<YOUR-REPOSITORY>.git
git push -u origin main
```

## Step 6: Deploy to GitHub Pages

MkDocs includes a magic command that builds your static site and automatically pushes it to a branch called gh-pages in your repository.

Simply run:

```bash
mkdocs gh-deploy
```

Congratulations! Your site should now be building.

## Final Configuration in GitHub

1. Go to your repository on GitHub.
2. Go to Settings > Pages (on the left sidebar).
3. Under the Build and deployment section, ensure the Source is set to "Deploy from a branch".
4. Under Branch, select the gh-pages branch and the / (root) folder.
5. Save the changes.

In a couple of minutes, your site will be publicly available at the URL you configured (https://<YOUR-USERNAME>.github.io/<YOUR-REPOSITORY>/).

## (Optional) Automate with GitHub Actions

If you want your site to update automatically every time you git push to the main branch, create a file in your project at .github/workflows/deploy.yml with the following content:

```yaml
name: Deploy MkDocs
on:
  push:
    branches:
      - main
jobs:
  deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-python@v5
        with:
          python-version: 3.x
      - run: pip install mkdocs-material
      - run: mkdocs gh-deploy --force
```

And that's it! You now have a robust, elegant, and freely hosted documentation system.