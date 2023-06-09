# storybook-builder-wds

[Storybook builder](https://storybook.js.org/docs/web-components/builders/overview) powered by [@web/dev-server](https://modern-web.dev/docs/dev-server/overview/).

## Frameworks

Currently we support Web Components via `storybook-web-components-wds`.

## Install

Install npm packages:

```bash
npm install storybook-builder-wds storybook-web-components-wds --save-dev
```

## Setup

Follow the [official storybook docs](https://storybook.js.org/) to setup storybook.

You can choose Web Components and another builder when asked by the init CLI command.
Later you can change the framework to `storybook-web-components-wds` in the storybook main configuration file.

If you are migrating from the `@web/dev-server-storybook` plugin, please read the [migration guide](#migration) below.

## Configuration

When you finish the setup you'll have a standard storybook folder with configuration files (`.storybook` by default) containing at least `main.js`.

### Activating the builder

Change the following in `main.js`:

- set `config` type to `import('storybook-web-components-wds').StorybookConfig`
- set the `framework.name` to `'storybook-web-components-wds'`

```js
// .storybook/main.js

/** @type { import('storybook-web-components-wds').StorybookConfig } */
const config = {
  ...
  framework: {
    name: 'storybook-web-components-wds',
  },
  ...
};

export default config;
```

### Configuring storybook

The `storybook-builder-wds` aims at compatibility with standard storybook features, e.g. `.storybook/preview.js`.
Follow the [official storybook docs](https://storybook.js.org/) for the configuration options.

### Configuring dev server

The builder implements the `storybook dev` command and comes with a default `@web/dev-server` configuration which adheres to browser standards and supports essential plugins of the storybook ecosystem.

#### Config file and wdsConfigPath

When storybook is loaded, the builder's default `@web/dev-server` config gets automatically merged with the project's config file (`web-dev-server.config.{mjs,cjs,js}`) if present.
This is needed to ensure that the same configuration is used for application, feature or design system you are building.

Optionally you can configure a different path to this file using `framework.options.builder.wdsConfigPath`, relative to CWD:

```js
// .storybook/main.js

/** @type { import('storybook-web-components-wds').StorybookConfig } */
const config = {
  ...
  framework: {
    name: 'storybook-web-components-wds',
    options: {
      builder: {
        wdsConfigPath: 'storybook-wds.config.mjs',
      },
    },
  },
  ...
};

export default config;
```

#### wdsFinal hook

Sometimes you might need to add storybook specific configuration for dev server, you can use the `wdsFinal` hook for this.

```js
// .storybook/main.js

/** @type { import('storybook-web-components-wds').StorybookConfig } */
const config = {
  ...
  framework: {
    name: 'storybook-web-components-wds',
  },
  async wdsFinal(config) {
    // add storybook specific configuration for @web/dev-server
    // e.g. "open" to go to a custom URL in the browser when server has started
    config.open = '/custom-path';
    return config;
  },
  ...
};

export default config;
```

The `config` parameter is a `@web/dev-server` config, make sure to use appropriate options and @web/dev-server plugins.
When using rollup plugins make sure to [convert them to @web/dev-server ones](https://modern-web.dev/docs/dev-server/plugins/rollup/).

### Configuring static build

The builder implements the `storybook build` command and comes with a default `rollup` configuration which adheres to browser standards and supports essential plugins of the storybook ecosystem.

#### rollupFinal hook

Sometimes you might need to add some extra configuration for the static build, you can use the `rollupFinal` hook for this.

```js
// .storybook/main.js
import polyfillsLoader from '@web/rollup-plugin-polyfills-loader';

/** @type { import('storybook-web-components-wds').StorybookConfig } */
const config = {
  ...
  framework: {
    name: 'storybook-web-components-wds',
  },
  async rollupFinal(config) {
    // add extra configuration for rollup
    // e.g. a new plugin
    config.plugins.push(polyfillsLoader({
      polyfills: {
        esModuleShims: true,
      },
    }));
    return config;
  },
  ...
};

export default config;
```

The `config` parameter is a `rollup` config, make sure to use appropriate options and rollup plugins.

## Migration

This guide assumes you are migrating from the `@web/dev-server-storybook` plugin and `@web/storybook-prebuilt` storybook 6 bundle.

Most of the [official migration guide](https://storybook.js.org/docs/web-components/migration-guide) applies to this migration too, with a few additions and exceptions.
Please read the official guide and follow it for manual migrations, but before running the automatic `upgrade` script read the sections below.

### Run upgrade script

The automatic `upgrade` script can't detect the `@web/dev-server-storybook` plugin and won't work as a result.
But with a simple workaround you can make it work.

First install old version of `builder-vite` and old storybook 6 renderer for the technology you used in the old setup, e.g. for Web Components it will be:

```bash
npm install @storybook/builder-vite@0.4 @storybook/web-components@6 --save-dev
```

Then proceed with the `upgrade` script and follow it's interactive process:

```bash
npx storybook@latest upgrade
```

Then [activate the builder](#activating-the-builder) in the main storybook configuration.

### Uninstall old packages

```bash
npm uninstall @web/dev-server-storybook @web/storybook-prebuilt --save-dev
```

### Install addons

The old storybook 6 bundle `@web/storybook-prebuilt` came with a few preconfigured addons.
In the new setup you'll need to install and configure them explicitly.

We recommend to install the following addons:

```bash
npm install @storybook/addon-essentials @storybook/addon-links --save-dev
```

Then register them in the storybook main configuration:

```js
// .storybook/main.js

/** @type { import('storybook-web-components-wds').StorybookConfig } */
const config = {
  ...
  addons: [
    '@storybook/addon-essentials',
    '@storybook/addon-links',
    ...
  ],
  ...
};

export default config;
```

### Rename "rollupConfig" => "rollupFinal"

For consistency with other similar hooks in the storybook ecosystem, including this builder's own `wdsFinal`, the rollup hook was renamed to `rollupFinal`.

### CLI

`@web/dev-server-storybook` was a plugin of `@web/dev-server`, therefore you used to run storybook via `wds` CLI.

With the introduction of [builders](https://storybook.js.org/docs/web-components/builders/overview) in storybook 7 this is no longer the case and you can make use of [storybook CLI](https://storybook.js.org/docs/web-components/api/cli-options).

Typically you'll use `storybook dev` and `storybook build` to start the dev server and create a static build for deployment:

```json
// package.json
{
  ...
  "scripts": {
    ...
    "storybook:start": "storybook dev -p 6006",
    "storybook:build": "storybook build --output-dir demo-storybook",
    ...
  },
  ...
}
```

### Usage of ESM

Storybook 7 supports ES modules for configuration files, so we recommend using standard syntax for imports and exports in `.storybook/main.js` and other configuration files.

### Storybook specific configuration

The usage of `web-dev-server.config.{mjs,cjs,js}` file for storybook specific configuration is not recommended.
You should use this file for your project dev server.

If you have a storybook specific configuration, move it to the [wdsFinal hook](#wdsfinal-hook) instead.

#### "port" option

The `port` configured in `web-dev-server.config.{mjs,cjs,js}` won't have any effect on the storybook, because storybook CLI runs it's own dev server.

To open storybook dev server on a certain port use the storybook CLI argument instead: `storybook dev -p XXXX`.

#### "open" option

The `open` URL configured in `web-dev-server.config.{mjs,cjs,js}` won't have any effect on the storybook, because it conflicts with the storybook auto-open behavior.

To open a non-default URL do the following:

- disabled open in storybook CLI, e.g. `storybook dev -p XXXX --no-open`
- configure `open` in [wdsFinal hook](#wdsfinal-hook)

## Contributing

Please read the [Contributing guide](CONTRIBUTING.md).
