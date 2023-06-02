# Deploy Vite app to GitHub Pages using GitHub Actions

## Video tutorial

[![Screen Shot 2022-05-15 at 18 03 44](https://user-images.githubusercontent.com/58401630/168482351-ee1ae441-31a4-48f7-b3f1-09296ae82818.png)](https://www.youtube.com/watch?v=MKw-IriprJY)

## Step-by-step instructions

### Scaffold a new Vite app and init git

```bash
# Create new Vite project using React template
npm create vite@latest vite-project -- --template react

# Install dependencies and start development server
cd vite-project
npm install
npm run dev
```

If the project is working fine, let's init a new git repository.

```bash
git init
git add .
git commit -m "init vite project"
```

* https://vitejs.dev/guide/#scaffolding-your-first-vite-project

### Create a new GitHub repository

Go to https://github.com/new and create a new repository.

❗️ Make sure **Public** is selected if you don't have a premium account. Otherwise, you won't be able to host your app using GitHub pages.
![Screen Shot 2022-05-15 at 16 19 34](https://user-images.githubusercontent.com/58401630/168477505-b3f2fc8f-a248-499c-9645-0c0d2bd5de35.png)

Once the repo is created, copy and paste the instructions similar to these to your terminal

```bash
git remote add origin git@github.com:sitek94/vite-deploy-demo.git
git branch -M main
git push -u origin main
```

### Create deployment workflow

Create a new file: `.github/workflows/deploy.yml` and paste the following code:

```yml
name: Deploy

on:
  push:
    branches:
      - main

jobs:
  build:
    name: Build
    runs-on: ubuntu-latest

    steps:
      - name: Checkout repo
        uses: actions/checkout@v3

      - name: Setup Node
        uses: actions/setup-node@v3

      - name: Install dependencies
        uses: bahmutov/npm-install@v1

      - name: Build project
        run: npm run build

      - name: Upload production-ready build files
        uses: actions/upload-artifact@v3
        with:
          name: production-files
          path: ./dist

  deploy:
    name: Deploy
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'

    steps:
      - name: Download artifact
        uses: actions/download-artifact@v3
        with:
          name: production-files
          path: ./dist

      - name: Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./dist
```

This workflow will run on every push to the `main` branch. It will first build the project, and then deploy it to GitHub pages.

### Test deployment workflow

Commit deployment workflow and push the changes to GitHub.

```bash
git add .
git commit -m "add deploy workflow"
git push
```

When you go, to [Actions](https://github.com/sitek94/vite-deploy-demo/actions) and click on the recent workflow, 
you should see that it failed, because of missing permissions:

![Screen Shot 2022-05-15 at 16 33 13](https://user-images.githubusercontent.com/58401630/168478218-93f9fda7-91ff-49fb-b96c-8aa5e682ef70.png)

### Ensure Actions have `write` permission

To fix that, go to [Actions Settings](https://github.com/sitek94/vite-deploy-demo/settings/actions), 
select **Read and write permissions** and hit **Save**:

<img width="844" alt="Screen Shot 2022-05-15 at 16 35 37" src="https://user-images.githubusercontent.com/58401630/168478314-c11c7c49-eeeb-411c-8351-9dd2b7423681.png">

Basically, our action is going to modify the repo, so it needs the _write_ permission.

Go back to [Actions](https://github.com/sitek94/vite-deploy-demo/actions), click on failed workflow 
and in the top-right corner click on **Re-run failed jobs**

![Screen Shot 2022-05-15 at 16 41 29](https://user-images.githubusercontent.com/58401630/168478612-3a129490-4191-4380-9eec-f51f31b77720.png)

After job run, you should be able to see a new branch `gh-pages` created in your repository.

![Screen Shot 2022-05-15 at 16 43 26](https://user-images.githubusercontent.com/58401630/168478674-fa8f5cd7-305d-469d-ae86-c7bb96bf232f.png)

### Enable GitHub pages

To host the app, go to [Pages Settings](https://github.com/sitek94/vite-deploy-demo/settings/pages), set **Source** to `gh-pages`, and hit **Save**.

![Screen Shot 2022-05-15 at 16 47 07](https://user-images.githubusercontent.com/58401630/168478837-08c139b5-4afd-4adc-9a0d-a8a8a740d859.png)

After a while your app should be deployed and be available at the link displayed in Pages Settings. If you want to follow the deployment process,
go to [Actions](https://github.com/sitek94/vite-deploy-demo/actions) and **pages-build-deployment** workflow:

![Screen Shot 2022-05-15 at 17 07 16](https://user-images.githubusercontent.com/58401630/168479786-c87c99e7-97e1-44f0-8bec-08483300b410.png)

Once deployment is done, visit the app at: `https://<YOUR_GITHUB_USER>.github.io/REPO_NAME`

### Fix assets links

You will see that something is not right, because instead of there is a blank screen. When you inspect it, you will see that some files were not found.

![Screen Shot 2022-05-15 at 17 11 42](https://user-images.githubusercontent.com/58401630/168479964-7a0c6b8e-3be5-4468-af24-a09e13d800b1.png)

This is happening, because of the subdirectory-like URL structure GitHub uses for Project Pages. Asset links are referencing the files 
in the domain root, whereas our project is located in `<ROOT>/vite-deploy/demo`. This is how the links should look like:

```
❌ Bad
https://sitek94.github.io/assets/favicon.17e50649.svg

✅ Good
https://sitek94.github.io/vite-deploy-demo/assets/favicon.17e50649.svg
```

Fortunately, there is a very easy fix for this. Add the following line in `vite.config.js`:

```diff
import { defineConfig } from 'vite'
import react from '@vitejs/plugin-react'

// https://vitejs.dev/config/
export default defineConfig({
   plugins: [react()],
+  base: '/vite-deploy-demo/'
})
```

Now, asset links will have a correct path, so commit the changes, push the code, wait for the deploy to finish and see it for yourself!

### Final 

![Screen Shot 2022-05-15 at 17 31 23](https://user-images.githubusercontent.com/58401630/168480802-95978d6c-2532-49f0-b118-6a313286d512.png)

## FAQ

- [How to setup custom domain?](https://github.com/sitek94/vite-deploy-demo-custom-domain)

## Resources

- [Vite](https://vitejs.dev/)
- [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/creating-a-github-pages-site#creating-your-site)
- [GitHub Actions](https://docs.github.com/en/actions)
