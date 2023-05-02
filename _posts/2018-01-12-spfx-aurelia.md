---
layout: post
title:  "Aurelia and SPFx"
author: magnus
categories: [ aurelia, spfx ]
image: assets/images/5.jpg
featured: false
hidden: false
---
# Building Office 365/SharePoint applications with Aurelia #
## Why care about Microsoft Office 365 anyway? ##
Microsoft has been very successful in moving customers to their cloud offering Microsoft Office 365. You have more than 100 million potential end users in the Microsoft cloud. All these users create, share, and collaborate on documents and information critical to their own success. They use Mac or PC on any web browser or any mobile device. The foundation is in place for building great business applications that helps these customers to be even more productive, smarter and more agile. Microsoft provides ways to add functionality in many ways and an API to reach and interact with the user’s data. It is now up to you and your customers to build smart, snappy and sleek applications that live and run on top of Microsoft Office 365.

## What are my options, how can it be done? ##
So now all you need is a way to build a great user experience and call back-end services. Since you are an expert on Aurelia, off course you know that it is the way forward.
There are several ways of adding functionality to SharePoint online, the collaboration offering in Office 365. For simplicity the four choices below are the ones you will care about.
1.	Full immersive add-in running on your own hosted web server
2.	JavaScript injected into the SharePoint page itself
3.	Modern Client side webparts built with SPFx
4.	Adding UI commands
Choice number 1,2 and 3 can be built with Aurelia. Adding UI commands, number 4, is only the means for opening your UI that is built as 1, 2 or 3.
In this post I will focus on number 3, building modern client side webparts with SPFx and Aurelia.

## SPFx and Aurelia ##
The SharePoint Framework (SPFx) is a page and web part model that provides full support for client-side SharePoint development, easy integration with SharePoint data, and support for open source tooling. This means that your code runs in the browser in the context of the user. It is easy to call the SharePoint API and it is easy to build a modern responsive experience.
The tooling you need on your local developer machine is Node, NPM and an editor of your choice. The code is written in TypeScript. SPFx is released to NPM and you use Yeoman to create your project. When you build your project, there is a gulp task that dynamically builds a WebPack configuration file that is then used by WebPack for building and bundling.
Fortunately, Microsoft encourage us to extend the build process so I have adapted the build process for us Aurelia developers. As always, my work stands on the shoulders of earlier work such as the Aurelia WebPack plugin.

## Getting started ##
If you are eager just to try it out, download the Aurelia Navigation Skeleton from my GitHub [https://github.com/magnusdanielson/spfx-aurelia]
It is important to verify your version of Node and NPM. These version dependencies are related to SPFx and not to Aurelia it-self.
```shell
npm --v
3.10.8

node -v
v6.11.5
```

When you have verified your versions, just install everything you need with:
```shell
npm install
```

You can then admire the beauty of Aurelia Navigation Skeleton in the SharePoint Workbench. Notice that the Office 365 top menu is hidden with the Bootstrap navigation menu. This is something you would never do in any real application but I wanted to stick as close as possible to the Aurelia Navigation Skeleton example, and there for I did not touch the css in any way.

## The details of Aurelia and SPFx ##
When you create your client side webpart with SPFx you get a complete projects structure with all the things you need to build and run the app. To use Aurelia in the webpart there are a few things you need to do. If you run the Yeoman template with all default choices and most importantly 'No JavaScript framework'

First add dependencies to the following packages:
```shell
npm install aurelia-bootstrapper@2.1.1 --save
npm install aurelia-fetch-client@1.1.2 --save
npm install bluebird@3.5.0 --save
 
```

Aurelia fetch is needed if you want to talk to any back-end server, and since you always want that, you should include it. Bluebird is there to polyfill Promise for IE and Edge. 
Then add developer dependencies to the following packages:
```shell
npm install aurelia-webpack-plugin@2.0.0-rc.5 --save-dev
npm install expose-loader --save-dev
```

The Aurelia-WebPack plugin is there to make sure the dynamic nature of Aurelia works with WebPacks loaders. There are some configuration you can dive into about the plugin but we won’t go into details about that now. We will save that for a later post.
Then add the following function to the gulp.js file that already is created in your project, just before the call to `build.initialize(gulp);`

```javascript
const { AureliaPlugin } = require("aurelia-webpack-plugin");

build.configureWebpack.mergeConfig({
    additionalConfiguration: (generatedConfiguration) => {
      //generatedConfiguration.resolve.modules.push("src");
      generatedConfiguration.module.rules[0].issuer = {
        // only when the issuer is a .js/.ts file, so the loaders are not applied inside templates
        test: /\.[tj]s$/i,
      };

      var rule1 = { test: /\.css$/i,issuer: [{ test: /\.html$/i }], use: "css-loader"} ;
      generatedConfiguration.module.rules.push(rule1)

      var rule2 = { test: /\.ts$/i, use: "ts-loader" };
      generatedConfiguration.module.rules.push(rule2);
      
      var rule3 = { test: /[\/\\]node_modules[\/\\]bluebird[\/\\].+\.js$/, loader: 'expose-loader?Promise' };
      generatedConfiguration.module.rules.push(rule3);

      
      generatedConfiguration.plugins.push(new AureliaPlugin(
      {
        aureliaApp: undefined
      }));
      
      return generatedConfiguration;
    }
  });
```

The MergeConfig function is called from the WebPack gulp task when you run for example gulp serve. It is our chance to inject our custom configuration before WebPack starts.

Add below to the tsconfig.json file in the `compilerOptions` section. I don’t really like that addition but that solves some problems we get from the TypeScript lint gulp task. I will work on that at a later time.

```javascript
    "skipLibCheck": true
```

Add the following imports to the HelloWorldWebPart.ts
```javascript
import { Aurelia } from 'aurelia-framework';
import { PLATFORM } from "aurelia-pal";
import * as Bluebird from 'bluebird';
```

Next add a constructor to the same file to configure Blubird as early as possible.
```javascript
public constructor()
  {
    super();

    Bluebird.config({ warnings: { wForgottenReturn: false } });


  }
```

Replace the render() function with below
```javascript
public render(): void {
     this.domElement.innerHTML = `
      <div id="${this.instanceId}" class="${this.instanceId}"  >Loading...</div>
      `;

      require(['aurelia-bootstrapper'],(au) =>
      {
        au.bootstrap(
          (aurelia: Aurelia) =>
          {
            aurelia.use
            .standardConfiguration()
            .developmentLogging();
            var el = document.getElementById(this.instanceId);
            aurelia.start().then(() => aurelia.setRoot(PLATFORM.moduleName('webparts/helloWorld/myapp'),el));
          }
        );

      });
  }
```

Notice that we just add one div tag and no Aurelia-app attribute. We do that so we can control how Aurelia starts the app. If we would just inject an Aurelia-app attribute it would work but we would get in all sorts of problems when the next webpart of the same type is added. We now control the start by requiring Aurelia-bootstrapper and then manually start the app. InstanceId is unique for every webpart added to the page, so we are sure there is no conflict between them.

Now add a simple myapp.ts file and a myapp.html file in the helloWorld folder.

```javascript
export class myapp
{
    public message:string = "World";
}
```

```html
<template>
    Hello ${message}
</template>
```

## Summary ##
Aurelia works great with modern SharePoint. My plan is to write more posts about both modern and old school SharePoint with Aurelia. Checkout [http://blog.dunite.se] for more posts or my twitter account @magnusdanielson.