# GitHub Pages Express.js Template

Build a static website with Express.js, then publish the generated `dist/` folder to GitHub Pages. The repository also includes a GitHub Codespaces development environment.

## 1. Open in GitHub Codespaces

Open the repository on GitHub, select **Code**, open the **Codespaces** tab, and create a codespace.

The development container provides Node.js 22. After adding your `package.json`, install the dependencies:

```bash
npm install
```

## 2. Add package.json

Create `package.json` in the repository root. The `build` script is required because the GitHub Actions workflow runs `npm run build`.

```json
{
  "name": "my-github-pages-site",
  "version": "1.0.0",
  "private": true,
  "scripts": {
    "build": "node server.js"
  },
  "dependencies": {
    "express": "^5.1.0"
  }
}
```

Do not remove or rename the `build` script unless you also update the workflow.

## 3. Add server.js

Create `server.js` in the repository root. This example uses Express routes to render pages, requests those routes during the build, and writes the returned HTML into `dist/`.

```js
const express = require("express");
const fs = require("node:fs/promises");
const path = require("node:path");

const app = express();
const outputDirectory = path.join(__dirname, "dist");
const publicDirectory = path.join(__dirname, "public");

app.get("/", (req, res) => {
  res.send("<!doctype html><html lang=\"en\"><head><meta charset=\"utf-8\"><title>Home</title></head><body><h1>Hello from Express.js!</h1><a href=\"/about\">About</a></body></html>");
});

app.get("/about", (req, res) => {
  res.send("<!doctype html><html lang=\"en\"><head><meta charset=\"utf-8\"><title>About</title></head><body><h1>About</h1><a href=\"/\">Home</a></body></html>");
});

const routes = ["/", "/about"];

async function build() {
  await fs.rm(outputDirectory, { recursive: true, force: true });
  await fs.mkdir(outputDirectory, { recursive: true });

  try {
    await fs.access(publicDirectory);
    await fs.cp(publicDirectory, outputDirectory, { recursive: true });
  } catch (error) {
    if (error.code !== "ENOENT") throw error;
  }

  const server = await new Promise((resolve, reject) => {
    const instance = app.listen(0, "127.0.0.1", () => resolve(instance));
    instance.on("error", reject);
  });

  try {
    const address = server.address();
    const baseUrl = `http://127.0.0.1:${address.port}`;

    for (const route of routes) {
      const response = await fetch(new URL(route, baseUrl));

      if (!response.ok) {
        throw new Error(`Failed to build ${route}: HTTP ${response.status}`);
      }

      const outputPath = route === "/"
        ? path.join(outputDirectory, "index.html")
        : path.join(outputDirectory, route.slice(1), "index.html");

      await fs.mkdir(path.dirname(outputPath), { recursive: true });
      await fs.writeFile(outputPath, await response.text(), "utf8");
    }
  } finally {
    await new Promise((resolve, reject) => {
      server.close(error => error ? reject(error) : resolve());
    });
  }

  process.stdout.write("Static website generated in dist/\\n");
}

build().catch(error => {
  process.stderr.write(`${error.stack || error}\\n`);
  process.exitCode = 1;
});
```

Add every page you want generated to the `routes` array. For example, add `"/contact"` when you create an Express route for that page. Each route is saved as an `index.html` file in its own folder, so `/about` becomes `dist/about/index.html`.

Put static assets such as CSS, browser-side JavaScript, fonts, and images in a `public/` folder. The build copies the contents of `public/` into `dist/`. Reference those assets from your pages using paths such as `/style.css` and `/images/logo.png`.

## 4. Install and build

Run these commands from the repository root:

```bash
npm install
npm run build
```

Make sure the `dist/` folder contains `index.html` and the other generated pages.

## 5. Deploy

Push your changes to the `main` branch. The workflow runs `npm install`, runs `npm run build`, and deploys `dist/` to GitHub Pages.

In the repository settings, open **Pages** and set the build and deployment source to **GitHub Actions**.

## Important

GitHub Pages only hosts static files. It does not keep an Express.js server running after deployment. Express is used during the build to generate HTML files; browser-side assets are copied into `dist/` and published as a static site.
