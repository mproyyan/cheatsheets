<h1 align="center">Vite</h1>

## Installation

```console
npm install --save-dev vite
```

or <br>

```console
npm install -D vite
```

## Entry Point

By default Vite will use index.html as the entry point but you can change it as in the code below

```javascript
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        app: "./main.html",
      },
    },
  },
  server: {
    open: "/main.html",
  },
});
```

## Run Vite

```console
npx vite
```

## Build

```console
npx vite build
```

During the Vite build process it will carry out dependency pre bundling, which is Vite's process of compiling project dependencies into one JavaScript file before the web application is loaded. The results of the build process by default will be saved in the `dist` folder or you can change it by configuring the `vite.config.js` file

```javascript
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    outDir: "my-build",
  },
});
```

## CSS Bundling

Vite automatically bundles CSS or CSS preprocessor connected to index.html. You can also use the `@import url("path-to-file")` syntax to import CSS files.

## Static Files

Vite supports bundling static files such as JSON files, images etc

## Public Directory

Public directories are directories used to store static asset files, such as images, fonts, CSS, and JavaScript. Files in a public directory can be accessed using an absolute URL, such as `/image.png`. This URL will remain the same, even if the web application is deployed to a different server. By default, the public directory is located in the public directory. This directory can be changed using the `publicDir` property in the Vite configuration file.

```javascript
import { defineConfig } from "vite";

export default defineConfig({
  publicDir: "customPublicDir",
});
```

**The following are some examples of using public directories** <br>

- Save images, fonts, and CSS for web applications
- Store JavaScript files that don't need to be bundled, such as JavaScript files for analytics or advertising.
- Save HTML files for static pages, such as about pages or terms and conditions.

## Preview

`npx vite preview` command is used to run a Vite project in production mode and view the results in the browser, while `npx vite build` command is used to build a Vite project for production and generate static files that can be deployed to a CDN server or other web server.

## Multi Page App

A multi-page web application (MPA) is a web application that consists of several interrelated pages. These pages are usually accessed via different URLs. To create an MPA, you need to create several HTML, CSS, and JavaScript files. You also need to add some configuration to the `vite.config.js` file.

```javascript
import { defineConfig } from "vite";

export default defineConfig({
  build: {
    rollupOptions: {
      input: {
        // list of pages you want to create
        index: "index.html",
        blog: "blog.html",
        contact: "about/contact.html",
      },
    },
  },
});
```

## Template

A Vite template is a preconfigured Vite project with everything you need to start a new project. These templates typically include basic HTML, CSS, and JavaScript files, as well as customized Vite configuration for a specific framework or library. Vite provides several built-in templates for popular frameworks and libraries, such as React, Vue, Svelte etc. To use the Vite template you can use the command below

```console
npm create vite@latest
```
