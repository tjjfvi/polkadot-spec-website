# Polkadot Spec Website

This repository contains the source code for the Polkadot specification website.

Specifically, all the Docusaurus configurations, plugins, scripts, and components used to build the website are located in this repository.

The main repository is [polkadot-spec](https://github.com/w3f/polkadot-spec).

You'll need to clone the [main repo](https://github.com/w3f/polkadot-spec) if you want to build the entire website locally.

## Documentation

Here you can find an explanation of how the website is built.

The macro scripts (`npm run build` and `npm run start`) are defined inside `package.json`.

They will call the bash scripts, located inside the `scripts` folder, and others in TypeScript located inside the `preBuild` folder.

The core thing to understand is that the website is built in two steps:
- First, the `preBuild` scripts will generate new markdown files, starting from the markdown files the user has written. The new ones will contain, for example, the rendered placeholders defined inside the initial ones. This step is needed because it's not possible to do so after the build or with custom Docusaurus plugins;
- Second, Docusaurus will build the website starting from the markdown files generated in the previous step.

On top of this, the generated HTML pages will be affected by the scripts inside `plugins`. 

Inside `src` and `static` there are custom components and static files used by the website.

### `scripts`

The bash scripts inside this folder are called by the `package.json` scripts.
- `copy_src.sh` safely copies all the files inside the parent folder of the `polkadot-spec-website` repo (which is `polkadot-spec`);
- `kaitai_render.sh` renders the Kaitai Struct files, generating the corresponding images;
- `tsc.sh` compiles the TypeScript files inside `preBuild` and `plugins`.

### `preBuild`

These scripts are called before the Docusaurus build.
- `addBibliographyTitle` looks for all the markdown pages that contain a bibliography citation and add the "Bibliography" title to them;
- `checkBrokenExternalLink` checks if there are broken external links inside the markdown files, and print a report in the console during the execution of the `npm run build` script;
- `graphvizSvgFixer` finds all the kaitai rendered SVG images and fixes them: removes useless attributes and adds links to other images (for this reason, if two images are on two different pages, the script needs to map every image to the corresponding page, and then it can add the links);
- `numerationSystem` is the most important one: it assigns numbers/letters to all the entities in the specification, such as definitions, sections, chapters, algorithms, images, and others. It finds the placeholders and they get replaced in the new markdown files.


### `plugins`

These scripts are called after the build or when the website pages are loaded in the browser.
- `addLocationChangeEvent` adds a custom event in the environment, which is triggered when the user changes the page;
- `checkBrokenInternalLink` checks if there are broken internal links inside the HTML files, and print a report in the console during the execution of the `npm run build` command;
- `fixAlgoCounters` removes the useless (and wrong) counters from the algorithms' pseudocode (generated by the component `src/components/Pseudocode.jsx`);
- `highlightBibLinks` finds the bibliography URLs which are not inside a `<a>` tag and fixes them;
- `injectGlossaryCss` injects CSS classes to the glossary elements so that they can be styled;
- `navbarActiveItem` adds/removes the `active` class to the navbar items, depending on the current page;
- `redirectOldLinks` redirects the old links to the new ones; this [community plugin](https://docusaurus.io/docs/api/plugins/@docusaurus/plugin-client-redirects) is not sufficient, as to redirect we must know where a section is located in the new website (e.g., the current section findable at https://spec.polkadot.network/chap-state#sect-state-replication was findable with https://spec.polkadot.network#sect-state-replication in the old website);
- `resizeSvg` resizes the SVG images when a page is loaded; this is needed because the `graphvizSvgFixer` sometimes removes useless things from the SVGs, and this can cause the images to be too small;
- `safePluginExecution.ts` makes the environment execute the plugins after `timeout_ms` milliseconds so that the website is always loaded before the plugins are executed.