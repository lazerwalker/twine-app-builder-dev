# Twine App Builder

![Build and Test](https://github.com/lazerwalker/twine-app-builder/actions/workflows/main.yml/badge.svg)

This project takes your existing browser-based game and automatically generates a standalone desktop version for both Windows and macOS.

It's primarily intended to take games made in tools like [Twine](https://twinery.org) (both 1 and 2), [Bitsy](http://www.bitsy.org/), and [PuzzleScript](https://www.puzzlescript.net/) and produce desktop builds suitable for distribution on platforms such as Steam or Itch. But you may find other uses for it as well! It works with any game whose output is HTML/JavaScript/CSS.

**And yes, it works with custom assets like images and audio!** If your game works in Google Chrome, it should work here. Your game can additionally work offline, as long as you don't reference any externally-hosted assets or scripts.

To use this, you will need basic familiarity with git and GitHub. No other technical expertise is needed beyond whatever you need to make your game!

For the technically-minded: this uses GitHub Actions to bundle up your game into a prewritten Electron app template, which then gets built for Windows and macOS.

## How to Use

More detailed instructions follow, but here's the big-picture of what you'll be doing:

1. Fork this git repo, and add your game's `index.html` file (and any art assets) to it.
2. When you commit those changes to git and push them to GitHub, GitHub automatically takes your game files and bundles them up into downloadable desktop binaries
3. Your GitHub repo's "Releases" section will now contain downloadable Windows and Mac versions of your game
4. Any time you push new code to your repo, this process will repeat and new binaries will be auto-generated!

## Getting Started

1. While logged in to GitHub and viewing this project, click the green "Use this Template" button at the top of the page.
2. Move your game files into your new git repository. Put anything you'll need into the `src` folder. This must include an `index.html` file, which will be loaded in a custom web browser whenever players open your game, but might also include other resources like images or audio. If you have files like images, audio, or external JavaScript, it's better to include them directly in this folder instead of linking to external URLs so your game will work offline.
3. In your new repo, there will be a file in the `.github/workflows` subfolder called `main.yml`. Down on [line 89](https://github.com/lazerwalker/twine-app-builder/blob/main/.github/workflows/main.yml#L89), in the "Build the app" section, change the `APP_NAME` variable from "My Twine Game" to whatever you want your app to be called.
4. If you have a custom app icon you'd like to use, put that as `icon.png` in the root of the repo. It will be automatically resized as long as it is square and at least 1024x1024.
5. Commit and push these changes to GitHub
6. Wait a few minutes! You can go to the "Actions" tab in your GitHub repo to see build progress. If you don't see any progress in the Actions tab, you may need to enable Actions for your repo in the repository settings.
7. When the build is done, the "Releases" section of your repo will contain download links. You can find that by clicking the "Releases" section on the right-hand side of your main repo page, or going directly to https://github.com/YOUR_USERNAME/YOUR_REPO/releases.

As you make changes to your game, repeat the last few steps. Every new git commit that you make and push up to GitHub will result in a new build of your game.

## Advanced Features

### Building less frequently

By default, this generates new builds every time you push a new git commit to the `main` branch to GitHub. This is simple and easy to use, but it might be tiresome and unnecessary if you're making a lot of rapid changes to your game.

For more mature projects, it's recommended that you make triggering builds a slightly more manual process. Here are two recommended paths that both require slightly more git knowledge:

#### Build by pushing to a different branch

Instead of generating binaries every time you push code to the main branch, one approach is to work off of a development branch, and merge those changes into a production branch every time you want a new build.

As mentioned, by default this GitHub Action runs every time code is pushed to `main`. An easy process change would be to push work-in-progress commits to a branch called `dev` (or similar), and treat `main` as your production branch. If you would prefer dev work to happen on `main`, you can create a separate branch intended for production builds (e.g. `prod`), and you just need to update the correct [GitHub Actions workflow line](https://github.com/lazerwalker/twine-app-builder/blob/main/.github/workflows/main.yml#L6) to trigger builds off of `prod` instead of `main`.

You will also need to make this change if your default git branch happens to be something other than `main`. For older existing projects, your main/default branch may be called `master`.

#### Build by pushing git tags (recommended for advanced users)

A common approach to software versioning is to use git's built-in tagging system to flag certain git commits as certain versions. A more advanced option for this tool is to tell GitHub Actions to build new desktop binaries only when you push a new git tag that matches a certain format. This is my personal preferred approach, as it makes it easier to maintain clear version numbers, but it does require more experience with git and version control.

At the top of the [GitHub Actions workflow file](https://github.com/lazerwalker/twine-app-builder/blob/main/.github/workflows/main.yml#L3-L6) lives some code that tells GitHub to run the Action whenever new code is pushed to `main`:

```
on:
  push:
    branches:
      - main
```

Instead, replace that in your own fork with a command to build any time a tag is created

```
on:
  push:
    tags:
      - 'v*'
```

Using the git command-line tool, you can create a tag pointing to the most recent git commit by typing `git tag TAG_NAME` (e.g. `git tag v1.0.3`). You can then push that tag to GitHub by running `git push --tags`, which (if you've made the previous change) will trigger a new build.

Windows version numbers can only contain numbers and periods, which means your version numbers must also conform to that. This project is hardcoded to check for a preceding 'v' (as in the YAML above), so you should be sure to make your git tags of the form `vX.Y.Z`.

### Customization

This project provides a sensible set of default options in the generated app, but you might want to customize it! If you want to add deeper integration with OS-level features, or make configuration changes to things such as the menu bar, you can modify the Electron app template being used to generate the app. This will require knowledge of JavaScript and Electron.

When GitHub Actions builds your app, it fetches a simple wrapper Electron app located at https://github.com/lazerwalker/electron-wrapper-template. You can fork that template, make any changes you want, and then update your GitHub Actions workflow to point to your own fork in the ["repository"]() key of the "Check out Electron app template" step.

### Windows signing

Signing your Windows app removes the "untrusted publisher" warning message that Windows may show upon running your game. If you're primarily publishing on Steam, this may not be necessary, as this warning does not show up for games launched via the Steam launcher. Signing your app for Windows requires purchasing a developer certificate.

**Warning: This has not been tested and may not work as-is**

1. Go through the process of [creating a certificate for package signing](https://docs.microsoft.com/en-us/windows/msix/package/create-certificate-package-signing?WT.mt_id=spatial-8466-emwalker).
1. Once you've done this and have a valid PFX file, base64 encode it. You can do this in PowerShell by using the command `certutil -encode infile outfile`.
1. Open up your GitHub repo's Action Secrets (Settings -> Secrets), and create two "Repository secrets". `CERTIFICATE_WINDOWS_PFX` should contain the base64-encoded contents of your PFX file, and `WINDOWS_PFX_PASSWORD` should contain the password.

### macOS Signing and Notarization

Signing and notarizing your macOS app will avoid warnings that your app is unsigned, which may require users to go into their security settings to allow it to run. At some point, Apple may _require_ all apps to be notarized, but that has not yet happened as of the writing of this README. Signing and notarizong your app requires a paid Apple developer account.

**Warning: This has not been tested and may not work as-is**

To notarize your app, set up two repository secrets (from your fork's repo page, Settings -> Secrets -> New repository secret) called `APPLE_ID` and `APPLE_ID_PASSWORD` containing the username and password (respectively) to an Apple ID that has the ability to notarize apps. You may want to create a dedicated free Apple developer user that has been granted access to your paid account instead of storing your personal Apple ID credentials.

Proper codesigning (instead of merely notarization) should be supported as well, but more work needs to be done here.

## How does this project work?

Under the hood, project relies on two core pieces of technology: [GitHub Actions](https://github.com/features/actions) and [Electron](https://www.electronjs.org/).

- GitHub Actions is a free service integrated with GitHub that can run little bits of code on cloud servers whenever you do things like push new code to GitHub.
- Electron is an open-source tool that lets you build desktop apps using browser technologies like HTML, JavaScript, and CSS

I maintain a GitHub repo that contains a minimal scaffolding project built on Electron. When new code is pushed in your repo, a GitHub Action runs that grabs your HTML files, injects them into that scaffolding project, and builds the project for you using Electron tools.

## "Why don't you support [insert feature here]?"

Open a GitHub Issue in this repo!

This project is an experiment, so I've intentionally kept the initial release very minimal. If people are actually using this, I'd love to expand on it! Let me know if you're using this and there's something you're dying to see added, or if there's some missing feature preventing you from using this, so I can prioritize improvements! Some specific things I'm currently thinking about:

- Linux support
- iOS and Android support
- More customization options that don't require forking the Electron template
- Integration with store platforms (e.g. Itch.io, Steam) to automatically upload new builds
- Game auto-updater (i.e. push out new versions of your game without requiring players to download new binaries)

## License

This project is licensed under the MIT license. See the `LICENSE` file in this repo for more info.
