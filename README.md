
# Migrating a Create React App (CRA) application to Vite

I had an existing app that was scaffolded using the Create React app (CRA) and extended with Craco. CRA didn't support the tooling I needed so I had to look for an alternative. I found Vite.

## Planning to upgrade to Vite

Vite is almost a drop-in replacement for CRA if you already use TypeScript. You will need to make some changes to your code but you should be able to just find and replace for most of those.

These are the steps we will follow to migrate.

1. Update your package.json
2. Add a Vite config
3. Update your tsconfig.json file
4. Install all the packages
5. Move your index.html file
6. Update the index.html contents
7. Update all your env vars

Let's go!

### 1. Update your package.json
Change your package.json scripts to use Vite. Don’t worry about not having it yet, we will install it later.

```
"scripts": {
    "start": "vite",
    "build": "tsc && vite build",
    "serve": "vite preview",
},
```
Make sure you delete react-scripts from your dependencies section.

Add some new devDependencies for Vite.
```
npm i vite @vitejs/plugin-react vite-plugin-svgr
```

### 2. Add a Vite config
Add vite.config.ts to the root of your project. I just use this basic configuration to start with.
```
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import svgrPlugin from 'vite-plugin-svgr';
import envCompatible from 'vite-plugin-env-compatible';

// https://vitejs.dev/config/
export default defineConfig({
	envPrefix: 'REACT_APP_',
	// This changes the out put dir from dist to build
	// comment this out if that isn't relevant for your project
	build: {
		outDir: 'build',
	},
	plugins: [
		react(),
		envCompatible(),
		svgrPlugin({
			svgrOptions: {
				icon: true,
			},
		}),
	],
	resolve: {
		alias: {
			'@': '/src',
		},
	},
	server: {
		port: 3000,
	},
});
```

### 3. Update your tsconfig.json
You must set the tsconfig.json to use esnext as a target, lib and module type. This might change in future versions of TypeScripts as more esnext functionality is added to the es standard for a given year so check the Vite site for updated config if this article is old.

Add the vite types to the types section. This tells TypeScript about the special Vite browser functionality that it provides for us.

Finally don't forget to set isolatedModules to true if you don't have that already. There is some modern Typescript functionality that is not supported by Vite yet.

```
{
  "compilerOptions": {
    "target": "ESNext",
    "lib": ["dom", "dom.iterable", "esnext"],
    "types": ["vite/client", "vite-plugin-svgr/client"],
    "allowJs": false,
    "skipLibCheck": false,
    "esModuleInterop": false,
    "allowSyntheticDefaultImports": true,
    "strict": true,
    "forceConsistentCasingInFileNames": true,
    "noFallthroughCasesInSwitch": true,
    "module": "esnext",
    "moduleResolution": "node",
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx"
  },
  "include": ["src"]
}
```

### 4. Install to update everything
Run yarn or **npm i** to install all the new dependencies we have added to the project.

### 5. Move your index.html file
Move the index.html from /public out to the root of the project.

Vite doesn't need the index.html to be in the public folder any more.

### 6. Update the content of index.html
Vite handles URLs in the index.html differently to create a react app.

Remove any **%PUBLIC_URL%** references from the file. Just replace that string with **""**.

```
{/* This is the create react app URL. change this to not have the variable... */}
<link rel="icon" href="%PUBLIC_URL%/favicon.ico" />

{/* ... to be like this. This is the correct URL for Vite */}
<link rel="icon" href="/favicon.ico" />
```

Add a script tag with the project entry point

```
<body>
  <noscript>You need to enable JavaScript to run this app.</noscript>
  <div id="root"></div>
  {/* Like below. This is the script tag for bootstrapping your Vite application */}
  <script type="module" src="/src/index.tsx"></script>
</body>
```

### 7. Update all your env vars if you use them
Rename your environment variables so they start with VITE_ e.g. search and replace REACT_APP_toVITE_

```
# this create react app variable
REACT_APP_MY_VAR

# should be this in Vite
VITE_MY_VAR
```

Vite uses the ECMAScript module import.meta properties to pass environment variables to the modules.

To access these environment variables you must change all **process.env.** s to **import.meta.env.**.

You should be able to search and replace this.

### Additional notes for large existing projects

Vite recommends using CSS modules for your application. You should install the Sass preprocessor if you use Sass or CSS.

```
npm i --save-dev sass
```
If you facing issue related to **@import** you need to replace **@import** to **@use**, [ref](https://sass-lang.com/documentation/breaking-changes/import/)

If you must have react or vue environment variables available in process.env for your Vite application then you can use the plugin [vite-plugin-env-compatible](https://www.npmjs.com/package/vite-plugin-env-compatible).

The plugin will load **VUE_APP_** or **REACT_APP_** environment variables to **process.env.** You might need this if you are using a library that expects an env var to be on process.env for example.

```
npm i vite-plugin-env-compatible
```
and add
```
envCompatible()
```
to your vite.config.ts plugins section.

### Done!
That’s it. Now try running your app with npm run start

Let me know if anything doesn’t work for you!
