# Lumy app builder

This repository contains a Github action for building Lumy desktop app (Electron).


## Build workflow structure

The [build action](.github/workflows/build-electron.yml) has been moved from the [Lumy repository](https://github.com/DHARPA-Project/lumy) into this repository to limit write access. This reduces the risk of accidentally sharing certificates and secrets used to code sign the application.

The build action is configured to check out [Lumy repository](https://github.com/DHARPA-Project/lumy) and work with its `master` branch. The action is triggered by tagging this repository with a version tag (prefix `v`, for example: `v0.1.0-2021-09-21`). The tag is used to extract the version number that will be used during the build process: the `v` prefix is removed and the rest of the tag is considered the version. The action then sets the extracted version as the version of the `applications/jupyter-app/package.json` file and starts the build process.

When the build process is finished, the application is code-signed for Mac and Windows platforms.

## Release process

### 1. Verify the master branch of Lumy

Make sure that the `master` branch of the [Lumy repository](https://github.com/DHARPA-Project/lumy) repository is ready to be released: the front end has been tested, the correct version of `lumy-middleware` is specified in the [Jupyter App package.json file](https://github.com/DHARPA-Project/lumy/blob/master/applications/jupyter-app/package.json) (`lumy.middleware.version` field) and this middleware version was released (see [Gemfury repository page](https://manage.fury.io/dashboard/dharpa/package/pkg_CGYjR/version/ver_jMDU6) and [Lumy Middleware documentation](https://github.com/DHARPA-Project/lumy-middleware) for release instructions).

### 2. Pick the version and tag the repository

Choose the version number and tag the repository with this version prefixed with `v`. While no public releases have been done yet we have been using semver versions with an ISO date suffix, e.g. `v0.1.0-2021-09-21`. This version structure guarantees that the latest version is always at the top of the list if versions are sorted alphabetically. Tagging the repository and pushing the tags: 

```shell
git tag v0.1.0-2021-09-21
git push origin --tags
```

### 3. Wait for the build to finish

Open the [Actions page](https://github.com/DHARPA-Project/lumy-build/actions) and check that the build process has started. It can take up to 15 minutes to complete and depending on the health of GitHub build servers it may not complete successfully the first time. If the build is not successful, it can be restarted by clicking on the failed build from the list and then selecting the `Re-run jobs` action in the top right corner of the build page. 

If the build still fails, it is worth checking if all the assets have actually been built during multiple runs. The build has three jobs - one per operating system (Mac, Win, Linux) and it is possible that every failed build run was a result of only one of these jobs failing. To check the assets, select the build from the [releases page](https://github.com/DHARPA-Project/lumy-build/releases) and check what assets are available. A complete build should have assets for all three operating systems:

- `.tar.gz` file for Linux (e.g. `@lumy.jupyter-app-0.1.1-2021-09-15-2.tar.gz`)
- `.dmg` file for Mac (e.g. `Lumy-0.1.1-2021-09-15-2.dmg`)
- `.exe` file for Win (e.g. `Lumy-Setup-0.1.1-2021-09-15-2.exe`)

If these files are in the list, the build is successful and the build can be made available for Lumy app auto updater.

### 4. Make the build available for Lumy auto updater

Lumy app uses [Electron auto updater](https://www.npmjs.com/package/electron-updater) to automatically update the app. The auto updater checks GitHub for new releases and allows the user to download and install them directly from the app.

To make the new version visible to the auto updater, we need to mark the build as `release` or `pre release`. Use `pre release` for all internal releases to hide it from the `releases` page on GitHub.

To mark a build as a `pre release`, select the build from the [releases page](https://github.com/DHARPA-Project/lumy-build/releases), click `Edit draft` in the top right corner, check the `This is a pre-release` at the bottom of the page and click the `Publish release` button. 

Once the new release is available it can be downloaded and installed by selecting `Check for updates` from the Lumy main menu.

If the app cannot find the release it is worth checking the application console logs by starting the app from the console and monitoring the logs for any errors. On Mac the app can be started with `/Applications/Lumy.app/Contents/MacOS/Lumy`.