Next.js is a React framework that helps you build full-stack web applications.

# Run the Node JS application locally

npm install (Install Dependecies)
npm run build (Code Compilation)
npm run dev (To run your Next.js application in development mode)
http://localhost:3000 (Access it)

# Deployment STAGED explained

This is the **`deploy` job** of a **GitHub Actions** workflow. Its purpose is to **build a Next.js application and deploy it to GitHub Pages**.

Let's go through it section by section.

---

## Job Definition

```yaml
deploy:
```

* Creates a job named **deploy**.
* This job will execute independently after its dependencies are satisfied.

---

## Job Dependency

```yaml
needs: test
```

* The `deploy` job will **only run if the `test` job completes successfully**.
* If the `test` job fails, GitHub Actions skips this deployment.

Example:

```
Build  ✅
   │
Test   ✅
   │
Deploy ✅
```

If Test fails:

```
Build  ✅
   │
Test   ❌
   │
Deploy (Skipped)
```

---

## Permissions

```yaml
permissions:
  contents: write
  pages: write
  id-token: write
```

These permissions grant the workflow access to specific GitHub resources.

### 1. contents: write

```yaml
contents: write
```

Allows the workflow to:

* Read repository contents
* Push commits
* Create releases
* Update files

Without this permission, the workflow cannot modify repository contents if required.

---

### 2. pages: write

```yaml
pages: write
```

Allows the workflow to deploy to **GitHub Pages**.

Without it, deployment will fail.

---

### 3. id-token: write

```yaml
id-token: write
```

Allows GitHub Actions to generate an **OpenID Connect (OIDC)** token.

GitHub Pages uses this token to verify that the deployment request is coming from a trusted workflow.

Think of it as:

```
Workflow
    │
Requests Identity Token
    │
GitHub verifies workflow
    │
Deployment allowed
```

---

## Environment

```yaml
environment:
  name: production
  url: ${{ steps.deployment.outputs.page_url }}
```

This defines the deployment environment.

### name

```yaml
name: production
```

The deployment will appear under the **Production** environment in GitHub.

Example:

```
Environments

Production
```

---

### url

```yaml
url: ${{ steps.deployment.outputs.page_url }}
```

After deployment, GitHub displays the deployed website URL.

For example:

```
https://username.github.io/project-name
```

**Note:** This references `steps.deployment.outputs.page_url`, so your workflow should include a deployment step with `id: deployment`. Otherwise, this output won't exist.

---

## Runner

```yaml
runs-on: ubuntu-latest
```

GitHub creates a fresh Ubuntu virtual machine.

Example machine:

```
Ubuntu Linux
Node
Git
npm
Docker
```

Every workflow run starts on a clean VM.

---

# Steps

Each step runs in order.

---

## Checkout Repository

```yaml
- name: checkout repo
  uses: actions/checkout@v4
```

Downloads your repository onto the runner.

Before:

```
GitHub Repository
```

After:

```
Runner

project/
    package.json
    next.config.js
    pages/
```

---

### Token

```yaml
with:
  token: ${{ secrets.GITHUB_TOKEN }}
```

Uses GitHub's automatically generated token.

It allows authenticated operations such as:

* Reading private repositories
* Pushing commits (if permitted)
* Creating tags

---

## Setup Node.js

```yaml
- name: use node.js
  uses: actions/setup-node@v4
```

Installs Node.js.

---

### Version

```yaml
node-version: '20.x'
```

Installs the latest Node.js 20 release.

Example:

```
Node 20.4
```

---

## Configure GitHub Pages

```yaml
- name: configure github pages
  uses: actions/configure-pages@v4
```

Prepares the environment for deployment to GitHub Pages.

It configures environment variables and settings required by GitHub Pages.

---

### Static Site Generator

```yaml
with:
  static_site_generator: next
```

Tells GitHub:

> "This project is built with Next.js."

GitHub applies the appropriate Pages configuration for a static Next.js export.

---

## Install Dependencies

```yaml
- run: npm install
```

Installs all dependencies from `package.json`.

Equivalent to running locally:

```bash
npm install
```

Example:

```
package.json
        │
        ▼
node_modules/
```

---

## Build Application

```yaml
- run: npm run build
```

Runs the build script defined in `package.json`.

Example:

```json
"scripts": {
  "build": "next build"
}
```

If using static export (for example, with `output: "export"` in your Next.js configuration), the build produces an `out/` directory containing static files.

Typical output:

```
out/

index.html
about.html
_next/
images/
```

---

## Upload Build Artifacts

```yaml
- name: upload artifacts
  uses: actions/upload-pages-artifact@v3
```

Uploads the generated static website so it can be deployed in a later step.

---

### Path

```yaml
path: './out'
```

Uploads everything inside the `out` directory.

Example:

```
out/

index.html
about.html
styles.css
images/
```

---

## Deploy Step

```yaml
- name: deploy
```

This step is **incomplete**. It only defines the step name and does not specify what action to execute.

A typical deployment step for GitHub Pages would be:

```yaml
- name: deploy
  id: deployment
  uses: actions/deploy-pages@v4
```

Here:

* `uses: actions/deploy-pages@v4` performs the actual deployment to GitHub Pages.
* `id: deployment` allows you to reference outputs such as:

```yaml
${{ steps.deployment.outputs.page_url }}
```

Without this action (and `id`), the deployment won't happen, and `page_url` won't be available.

---

## Overall Workflow

```text
          test Job
              │
      (must succeed)
              │
              ▼
        Deploy Job Starts
              │
              ▼
      Checkout Repository
              │
              ▼
     Install Node.js 20
              │
              ▼
   Configure GitHub Pages
              │
              ▼
       npm install
              │
              ▼
      npm run build
              │
              ▼
    Generate out/ folder
              │
              ▼
 Upload out/ as Artifact
              │
              ▼
 Deploy to GitHub Pages
              │
              ▼
https://username.github.io/repository
```

### One issue to fix

Your current workflow is missing the actual deployment action. Replace the last step with:

```yaml
- name: deploy
  id: deployment
  uses: actions/deploy-pages@v4
```

This completes the deployment process and makes `steps.deployment.outputs.page_url` available for the `environment.url` setting.
