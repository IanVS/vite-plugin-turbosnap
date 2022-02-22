# vite-plugin-turbosnap

[![NPM version](https://img.shields.io/npm/v/vite-plugin-turbosnap?color=a1b858&label=)](https://www.npmjs.com/package/vite-plugin-turbosnap)

Generate a `preview-stats.json` file from your [vite storybook](https://github.com/eirslett/storybook-builder-vite) project for consumption by [chromatic cli](https://github.com/chromaui/chromatic-cli) to support [turbosnap](https://www.chromatic.com/docs/turbosnap). Turbosnap can limit the number of files that are snapshotted by Chromatic to only those which might have changed due to the files that have changed. It is also smart enough to run all of the snapshots if certain files, like storybook config, package.json, or anything imported by preview.js has changed.

_This is experimental, and may not support all project and storybook configurations, yet._

Please open an issue if you experience any trouble, and be sure to include the log file that is generated from the `--diagnostics` flag of chromatic-cli, as well as the cli's output using the `--debug --trace-changed=expanded` flags.

## Installation

```bash
npm i -D vite-plugin-turbosnap
```

Add this plugin to `viteFinal` in your `.storybook/main.js`:

```ts
// .storybook/main.js

const turbosnap = require('vite-plugin-turbosnap');

module.exports = {
  core: { builder: 'storybook-builder-vite' },
  async viteFinal(config, { configType }) {
    if (configType === 'PRODUCTION' && outputDir) {
      config.plugins.push(turbosnap({ rootDir: config.root }));
    }

    // ...And any other config you need to change...

    // return the customized config
    return config;
  },
```

## Usage

When you run `build-storybook` to create your production storybook, an additional file will be created in the output directory, named `preview-stats.json`. See the [chromatic turbosnap docs](https://www.chromatic.com/docs/turbosnap) for more information on how to configure and use turbosnap.

## Prerequisites

- A [Chromatic](https://www.chromatic.com/) account.
- chromatic-cli [6.5.0](https://github.com/chromaui/chromatic-cli/blob/main/CHANGELOG.md#650---2022-02-21) or higher.
- storybook-builder-vite 0.1.17 or higher.

## How it works

The turbosnap feature of Chromatic limits the snapshots that are executed to only those which can be reliably traced back to a changed file. In order to do this, it needs to have a dependency graph of the project, so that it knows when you change `button.js`, for instance, that it needs to re-run not only the `button.stories.js` file, but also any other stories which might include your custom button component. By default, it relies on the `stats` file that is output from webpack, the default storybook builder. In order to use this feature with a vite project, this plugin collects information about each module being built by vite, constructs a mapping between each file and all of the files which import it, and generates a file that mimics that created by webpack, with only the information that the chromatic cli needs to perform its checks.

## Credits

Thanks to [Gert Hengeveld](https://github.com/ghengeveld) for guidance on chromatic cli changes, and to my team at [Defined Networking](https://www.defined.net) for giving me the time to build and test this on our own chromatic storybook project.
