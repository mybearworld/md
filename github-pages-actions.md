# Putting apps on GitHub Pages via Actions

While the UI says it's possible, nowhere is it stated how exactly one should go about uploading a GitHub Pages site via Actions.

This can be useful when using a node.js framework or library, which is what this file is focused on. It shouldn't be difficult to adapt this to different applications, though.

1. **Create a deployment key**<br>
   Go to <https://github.com/settings/tokens>, and create a repository token for your repository. Make sure to **not use the Beta feature** and assign the `repo` permission.
2. **Add the deployment key**<br>
   Go to your repository settings at **Security** > **Secrets and variables** > **Actions**. Create a new repository secret called `PERSONAL_TOKEN` and add the personal token you created above.
3. **Create the GitHub action**<br>
   Add the following file as **.github/workflows/gh-pages.yml**. It will set up NPM, PNPM, use `pnpm build` and expect the folder to be `/dist` - this is easily changeable, though.

   ```yml
   name: Upload to GitHub pages

   on:
     push:
       branches:
         - master

   jobs:
     build-deploy:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v1

         - name: Setup Node
           uses: actions/setup-node@v1
           with:
             node-version: "18.x"

         - name: Set up PNPM
           uses: pnpm/action-setup@v2
           with:
             version: 8

         - name: Cache dependencies
           uses: actions/cache@v1
           with:
             path: ~/.pnpm
             key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
             restore-keys: |
               ${{ runner.os }}-node-

         - run: pnpm install
         - run: pnpm run build

         - name: Deploy
           uses: peaceiris/actions-gh-pages@v3
           with:
             personal_token: ${{ secrets.PERSONAL_TOKEN }}
             publish_dir: ./dist
   ```
