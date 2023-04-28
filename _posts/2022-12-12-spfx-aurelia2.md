---
layout: post
title:  "SPFx Aurelia2"
author: magnus
categories: [ spfx, aurelia ]
image: assets/images/2.jpg
---
Aurelia has been around for a long time and think it truly is the best frontend framework out there. The second version of Aurelia is soon to be released and this guide provides the instruction on how to use it in SPFx for build SharePoint webparts.

## Start with a regular project

Begin by creating a new SPFx project with yeoman. Use the following command.

```c
yo @microsoft/sharepoint --framework minimal --component-type webpart --solution-name app --component-name HelloWorld
```

## Add configuration with aurelia-spfx yeoman generator

Next, modify the project with the Aurelia-SPFx yeoman generator. Install the generator using the following command if you haven't already.

```c
npm install aurelia-spfx -g
```

Run the generator in the same folder where the regular SharePoint generator was run. Answer "yes" to all questions.

```c
yo aurelia-spfx
```

## Try it out

To test the setup, open a terminal in the root of the project and run:

```c
gulp serve
```

Add the web part in the workbench to verify that the regular build still works. Then open another terminal and run:

```c
gulp buildwp5 --watch
```

Now refresh the browser and you should see your webpart built with Webpack 5. From now on, make changes to your webparts in the "srcau"-folder, just let the "src"-folder be unchanged.

## How does it work

First we can look at the code in the webparts render method.

```javascript
public async render() {
    // This line renders the html on the page2
    let id = this.instanceId;
    this.domElement.innerHTML = `<hello-world-my-component id="${id}"></hello-world-my-component>`;

    try {
      let id = this.instanceId;
      var au = new Aurelia();
      const rootContainer = au.container;
      rootContainer.register(
        Registration.instance("WebPartContext", this.context),
        Registration.instance("WebPartProperties", this.properties)
      );
     
      await au.register(<any>HelloWorldOtherStuff);

      au.app({ host: document.getElementById(id), component: HelloWorldMyComponent })
      .start();
    }
    catch (error) {
      console.log(error);
    }
  }
```

We set the inner html to the root component, in this case hello-world-my-component. We set the id to the instance id of the webpart too.

The try-catch is a normal Aurelia startup according to the documentation. The only this we added is the web part context and the web part properties are registered by name. That makes it very easy to get them with dependecy injection. We also register the `HelloWorldOtherStuff` component so it can used easily, the alternative is to `<require>`. I will explain that later.

The hello-world-my-component looks like this. Standard Aurelia component.

```javascript
import {inject} from "aurelia";
import { WebPartContext } from '@microsoft/sp-webpart-base';
import { IHelloWorldWebPartProps } from '../HelloWorldWebPart';
import * as strings from 'HelloWorldWebPartStrings';
@inject(Element, "WebPartContext","WebPartProperties")
export class HelloWorldMyComponent
{
    constructor(private element:Element, private context:WebPartContext, private properties:IHelloWorldWebPartProps )
    {
        // element.addEventListener('build', (theme) => { this.onThemeChange(theme); }, false);
        this._environmentMessage = this._getEnvironmentMessage();
        this.imageUrl = this._isDarkTheme ? require('./welcome-dark.png') : require('./welcome-light.png');
    }
    private _isDarkTheme: boolean = false;
    private _environmentMessage: string = '';
    public message = 'Hello World!' + strings.DisplayGroupName;
    private imageUrl;

    private _getEnvironmentMessage(): string {
      if (!!this.context.sdks.microsoftTeams) { // running in Teams
        return this.context.isServedFromLocalhost ? "AppLocalEnvironmentTeams" : "AppTeamsTabEnvironment";
      }
  
      return this.context.isServedFromLocalhost ? "AppLocalEnvironmentSharePoint" : "AppSharePointEnvironment";
    }

}
```

The html part of the component is also standard Aurelia style. It also demonstrates how to `<require>` another component and also use the hello-world-other-stuff we registered in the startup.

```html
<require from="./hello-world-more-stuff"></require>
<require from="./my.css"></require>
<section class="helloWorld ${context.sdks.microsoftTeams ? styles.teams : ''}">
    <div class="welcome">
        <img alt=""
            src="${imageUrl}"
            class="welcomeImage" />
        <h2>Well done, ${context.pageContext.user.displayName}!</h2>
        <div>${_environmentMessage}</div>
        <div>Web part property value: <strong>${properties.description}</strong></div>
        <div class="my">Message from hello-world-my-component is ${message}</div>
    </div>
    <div>
        <hello-world-more-stuff></hello-world-more-stuff>
        <hello-world-other-stuff></hello-world-other-stuff>
    </div>
</section>
```