---
layout: post
title:  "Using Webpack 5 when building spfx"
author: magnus
categories: [ spfx, sharepoint, webpack ]
image: assets/images/16.jpg
---
Currently SPFx is using Webpack 4 and that is causing some trouble for me when I depend on Webpack 5. The added benefit of this change is that your code builds lightning fast compared to the regular build system.

There are a few things we need to fix in the solution to make this work. You can adapt the fixes to your liking. I wanted to have all code in the same project and any assumptions are built upon that.

## Start with a regular project

First we create a new SPFx-project with yeoman.

```c
yo @microsoft/sharepoint --framework minimal --component-type webpart --solution-name app --component-name HelloWorld
```

## Add configuration with aurelia-spfx yeoman generator

Then we modify the project with help of another Yeoman generator. First install it if you have not already.

```c
npm install aurelia-spfx -g
```

Then we run the generator in the same folder as the regular SharePoint generator was run. Answer yes to all questions.

```c
yo aurelia-spfx
```

## Try it out

Now let's try this out. Detailed description of the complete setup follows after this. Open a terminal in the root of the project and start it is you always do.

```c
gulp serve
```

Open the workbench and add the webpart just to verify that the regular build is still working. Then open another terminal and execute below command.

```c
gulp buildwp5 --watch
```

Now refresh the browser and you should see your webpart built with Webpack 5. From now on, make changes to your webparts in the "srcau"-folder, just let the "src"-folder be unchanged.

## Detailed description

### Background

To make this work we keep the regular build in the src folder. The reason for this is that there are some Webpack plugins that creates the basic files to actually run the webpart, the manifest file is one example. I have tried to adjust the plugins to work with Webpack 5 but that is quite hard with out the source code and I think there is a limit on how much one can tinker with the files in the node_modules folder before it gets to crazy. There are dependencies that forces us to use Webpack 4 at this moment. I have asked Microsoft if they can provide more information on the plugins so that I can reuse them in this setup but I think we just have to wait for them to upgrade the plugins. One option is to recreate the plugins from scratch but I think there is too much work and hopefully Webpack 5 support from Microsoft will come in a not too distant future.

### Modifications to the project

If you look at the package.json file, you will notice that there are som changes, most important webpack@5.75.0. I have also added som webpack loaders and Aurelia. There is nothing in this project that forces you to use Aurelia, but that was the end goal for me. Just remove all references to Aurelia if you want to do no framework or React.

The gulpfile is modified so that wp5gulpfile.js is included. This change adds some new gulp tasks:
-copywp5
-buildwp5
-addlib
-bundlewp5

A new file au-tsconfig.json is also added. It contains the typescript configuration used when building with Webpack 5. If you have any specific changes you previously added to tsconfig it should now be entered to this file. Since we now also don't have any dependencies on the core build from SPFx we can build with any version of TypeScript. We are using typescript@5.0.2 in this project. If you want to play with new features from modern Typescript aswell, that should work fine.

An new folder "srcau" is also created. This where you build you Webpack 5 webparts. The yeoman generator adds all the necessery files from the webpart you selected. There is a new "libraries" folder that contains the "common" folder. The purpose of this is to have webpack create a specific bundle with common framework dependent files. To be specific, it does not make sense to include "Aurelia" in every webpart we add, it makes more sense to let webpack put all non changing framework code in a common bundle and just put the webpart code in its own bundle. Also note that it would be possible to use the library spfx type, but I find it handy to have a common bundle in all projects. See webpack-config.js for more details about bundles.

The WebPart.ts file has the code necessery to start Aurelia in the render method. If you want to use any other framework, just insert your code here. The "components" folder is just an example on how to do common tasks with Aurelia. Another blogpost will explain this in more detail.

Webpack-config.js is just a regular Webpack 5 build configuration file. It has an entry block. Note the "dependsOn" property that points to the "common" property.

```json
 entry: {
      'hello-world-web-part':
      {
        'import':'./srcau/webparts/helloWorld/HelloWorldWebPart.ts',
        dependOn: 'common',
      },
      'common': 
      {
        'import':'./srcau/libraries/common/common.ts',
 
      },
    },
```

There is a watch configuration if you used the --watch option when you run the gulp command.

```json
  watch: options.watch,
    watchOptions: options.watch ? {
      ignored: '**/node_modules',
      aggregateTimeout: 600,
    } : undefined,
```

The loaders part handles css and scss files properly. It also handles html and ts files. Images and fonts are served as assets.

### New gulp tasks

buildwp5 executes the Webpack 5 build according the configuration in webpack-config.js. It should then be followed by the new copywp5 task. That is taken care of in the bundlewp5 task.

There are som options to use
--analyze creates the information you need to what sizes packages have in your bundle
--watch turns on watch mode, so any changes will rebuild. Note that it is fast as lightning thanks to Webpack 5.
--production changes how the build is performed, use this if you want to bundle the webpart for release.

The copy task replaces the files created by the regular spfx bundle command. That means that you can never run the webpack 4 build at the same time as the webpak 5 build. Also note that the "src" folder should not be updated, just code in the "srcau"-folder.