---
title: React Integration with Stencil
sidebar_label: React
description: React Integration with Stencil
slug: /react
contributors:
  - jthoms1
  - adamdbradley
  - kensodemann
  - ErikSchierboom
  - brentertz
  - danawoodman
  - a-giuliano
  - rwaskiewicz
---

# React Integration

**Supports: React v16.7+ • TypeScript 3.7+ • Stencil v2.9.0+**

Stencil provides a wrapper for your custom elements to be used as first-class React components. The goal of a wrapper is to easily integrate your Stencil components into a specific framework. Wrappers provide a function that you can use within Stencil’s Output Targets to automatically create components for the targeted framework that wrap the web components you create in a Stencil project.

One benefit of the wrapper pattern includes improved maintainability since you can write code once, and reuse it across different frameworks. Today, there are some challenges associated with using HTML Custom Elements in a React app. Custom events are not handled properly, as well as properties/attributes that are not a string or number. By using Stencil's component wrappers, you can solve these issues and receive first-class React components.

## Setup

### Project Structure

To organize the generated component libraries for different frameworks, we recommend using a monorepo structure. This monorepo will contain your Stencil component library as well as the component libraries for whatever frameworks you choose. The overall structure of a monorepo with Stencil and React component libraries might look something like this

```
top-most-directory/
├── stencil-library/
│   ├── stencil.config.js
│   └── src/components/
└── react-library/
    └── src/
        ├── components/
        └── index.ts
```

To do this, start by creating a monorepo

```bash
mkdir {the name of your monorepo}
```

And then move your Stencil component library into your monorepo

```bash
mv {the path to your Stencil component library} {the path to your monorepo}
```

### Create a React Component Library

Next, we will need to create the React component library that will wrap your Stencil components. This library will be a sibling to your Stencil component library. Inside your monorepo, you can create your own React project, or you can use the [React component library template](https://github.com/ionic-team/stencil-ds-react-template) to bootstrap it. To do this, run the following command

```bash
git clone https://github.com/ionic-team/stencil-ds-react-template component-library-react
cd component-library-react
npm i
```

:::note
If you want to name your React component library something different, add the new name at the end of the clone command like so
:::

```bash
git clone https://github.com/ionic-team/stencil-ds-react-template {the name of your React component library}
cd {the name of your React component library}
npm i
```

If you do rename your React component library, be sure to change the `name` in the `package.json` to match your new name.

### Install the React Output Target in your Stencil Component Library

Now that the project structure is set up, we can install the React Output Target package in your Stencil component library. This package contains the React wrapper function that we will use to generate our React wrapped components inside a 'React component library'. To install the React Output Target package, run the following command in your **Stencil project directory**:

```bash npm2yarn
npm install @stencil/react-output-target --save-dev 
```

### Add the React Wrapper Function to your Stencil Component Library

With the React Output Target package installed, we can now configure our Stencil component library to build our React wrapped components (within our React component library). In the `stencil.config.ts` file of your Stencil component library, add the React wrapper function

```tsx
import { Config } from '@stencil/core';
import { reactOutputTarget as react } from '@stencil/react-output-target';

export const config: Config = {
  ...
  outputTargets: [
    react({
      componentCorePackage: 'your-stencil-library-name',
      proxiesFile: '../your-react-library-name/src/components/stencil-generated/index.ts',
      includeDefineCustomElements: true,
    }),
    {
      type: 'dist',
      esmLoaderPath: '../loader',
    },
    {
      type: 'dist-custom-elements',
    },
    ...
  ],
};
```

First, make sure to import the `reactOutputTarget` function from `@stencil/react-output-target` at the top of the file. With that imported, we can now use it in the `outputTargets` array and specify the relevant parameters. The `componentCorePackage` should be the name of your Stencil component library. The `proxiesFile` is the file that gets generated by the React wrapper function and contains the definitions of all the React wrapper components. Be sure to change the path names to reflect the names of your packages.

For details on the `includeDefineCustomElements` option, and all other options, visit the API documentation section below.

With the `reactOutputTarget` configured, we can now generate our React wrapped components. In your **Stencil component library** run

```bash npm2yarn
npm run build
```

You’ll see the new generated file in your React component library at the location specified by the `proxiesFile` .

```
top-most-directory/
├── stencil-library/
│   ├── stencil.config.js
│   └── src/components/
└── react-library/
    └── src/
        └── components/
        │   └── stencil-generated/
        │       └── react-component-lib/
        │       └── index.ts <-- the newly generated file
        └── index.ts
```

### Add the Components to your React Component Library's Entry File

:::note
If you are using our React template, this should already be prepared for you, and this step can be safely skipped.
:::

In order to make the generated files available within your React component library and its consumers, you’ll need to export everything from within your entry file - commonly the `src/index.ts` file. To do this, you’ll write:

```tsx
export * from './components';
```

### Link Your Packages (Optional)

If you want to build and test your components locally, you will need to link the packages together. This is a replacement for publishing packages to npm that allows you to develop and test locally. To do this, we’ll use the `npm link` command. This command creates a global symlink for a given package and thereby allows it to be consumed by other packages in your environment.

First, build your Stencil component library. In your **Stencil component library**, run

```bash npm2yarn
npm run build
```

Now you can create a symlink for your Stencil component library. From within your **Stencil component library**, run

```bash npm2yarn
npm link
```

With the symlink created, we next need to specify which packages will be consuming them. Your React component library will need to consume your Stencil component library. In the directory of your **React component library** run

```bash npm2yarn
npm link {Stencil library name}
```

And with that, your component libraries are linked together. Now, you can make changes in your Stencil component library and run `npm run build` to propagate them through to the React component library without having to relink.

:::note
As an alternative to `npm link` , you can also run `npm install` with a relative path to your Stencil component library. This strategy, however, will modify your `package.json` so it is important to make sure you do not commit those changes.
:::

## Usage

If you are developing and testing your React component library locally, you'll have to use `npm link` again to make your React component library available in your React application. If your components are published to npm, you can skip this step.

To link your React component library, navigate to your **React component library** and run

```bash npm2yarn
npm run build
npm link
```

To build your React component library and create a symlink to the project.

Navigate to your **React application directory** and run

```bash npm2yarn
npm link {React component library}
```

To make use of your React component library in your React application, import your components from your React component library in the file where you want to use them.

```tsx
// if your React component library has another name, replace 'component-library-react' with that name
import { MyComponent } from 'component-library-react';
```

With that, your component is now available to be used like any other React component.

## FAQ's

### What is the best format to write event names?

Event names shouldn’t include special characters when initially written in Stencil. Try to lean on using camelCased event names for interoperability between frameworks.

### How do I add IE11 or Edge support?

If you want your custom elements to be able to work on older browsers, you should add the `applyPolyfills()` that surround the `defineCustomElements()` function.

```tsx
import { applyPolyfills, defineCustomElements } from 'test-components/loader';

applyPolyfills().then(() => {
  defineCustomElements();
});
```

## API

### proxiesFile

This parameter allows you to name the file that contains all the component wrapper definitions produced during the compilation process. This is the first file you should import in your React project.

### includeDefineCustomElements

If `true`, React components will import and define elements from the [`dist-custom-elements` build](/custom-elements), rather than [`dist`](/distribution).

### excludeComponents

This lets you exclude wrapping certain Web Components. This is useful if you need to write framework-specific versions of components. In Ionic Framework, this is used for routing components, like tabs, so that Ionic Framework can integrate better with React's Router.