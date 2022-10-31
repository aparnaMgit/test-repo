

1. Create `remotes/dev` folder (it is in `.gitignore`)

2. Create `.json` files and point URLs to your locally served applications.
   (Look into files in `dev-example` folder for an example.)
   - `mfd-common-remote-entry.json` - contains URL to the _mfd-common_ micro-app's remote entry point.
   - `remote-entries.json` - contains remote entry URLs of every other micro-app

    **Note:** There's no need to specify all micro-app URLs. By default, they are falling back
    to the URLs in `dev-ga` folder that points to the Dev build of microfrontends
    with latest changes in their `main`/`master` branches.

3. Start the **mockServer**
   ```sh
   npm run start:mockServer
   ```
   If you open http://localhost:4480 in your browser, the message
   _"Hello from an Express mock server."_  should appear.


5. Start the **mfd-shell** micro-app's dev server
   ```sh
   npm start
   ```
