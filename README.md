# GitHub Pages JavaScript Template

A simple GitHub Pages template that builds your website with `npm run build` and publishes the contents of the `dist/` folder.

## 1. Add your website files

Put your website files in the repository root. For example:

```text
index.html
style.css
script.js
```

Add any other files your website needs, such as images or additional JavaScript and CSS files.

## 2. Create package.json

Create a file named `package.json` in the repository root and add this build script:

```json
{
  "scripts": {
    "build": "node -e \"const fs=require('fs');fs.rmSync('dist',{recursive:true,force:true});fs.mkdirSync('dist',{recursive:true});for(const f of ['index.html','style.css','script.js'])fs.copyFileSync(f,'dist/'+f)\""
  }
}
```

The filenames inside the array must match the files you want to publish. Add every required file to that array. For example, if you have `logo.png`, include `'logo.png'` in the array.

The build script clears and recreates `dist/`, then copies the listed files into it. The workflow publishes that folder.

## 3. Test the build

Run this command from the repository root:

```bash
npm run build
```

Check that the `dist/` folder was created and contains your website files. If a listed file does not exist, the build will fail.

## 4. Publish with GitHub Pages

Push your changes to the `main` branch. The GitHub Actions workflow runs `npm run build` and deploys the resulting `dist/` folder.

In your repository settings, open **Pages** and set the build and deployment source to **GitHub Actions**.

## Important

GitHub Pages hosts static websites. It does not run a persistent Node.js or Express server. Your `server.js` can be used as part of a build process if it generates static files, but the deployed website must consist of static files in `dist/`.
