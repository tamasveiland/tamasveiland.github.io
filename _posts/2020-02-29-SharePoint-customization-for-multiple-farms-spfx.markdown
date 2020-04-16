---
layout: post
title:  "Generate extensions for multiple SharePoint farms (modern experience)"
date:   2020-02-29 11:06:21 +0100
categories: 
    - "SharePoint"
---
This is the second part of the series that I started for describing SharePoint customization creation for multiple farms. If you haven't read the first post, I recommend to do so:
[Generate extensions for multiple SharePoint farms (classic experience)](../10/SharePoint-customization-for-multiple-farms-classic.html)

In this post, I'll elaborate on how to differentiate in image loading if you're building SharePoint Frameworks (SPFx) extension for multiple farms. The main reason for using this approach is the requirement to display different logos in the site header on certain SharePoint farms.

In case of SPFx, you will need to extend your gulfile.js file. The following code snippet shows how to propagate input variables to webpack:
```
'use strict';

const gulp = require('gulp');
const build = require('@microsoft/sp-build-web');
const webpack = require('webpack');
const getLogger = require('webpack-log');
const log = getLogger({ name: 'webpack-logger' });
 
build.configureWebpack.mergeConfig({
    additionalConfiguration: (generatedConfiguration) => {

        var config = build.getConfig();
        var p = config.args['process'];
        var farm = p.env.farm;
        log.info(`SharePoint farm to target: ${farm}`);
        generatedConfiguration.plugins.push(
            new webpack.DefinePlugin({'process.env.FARM': JSON.stringify(farm)}));

        return generatedConfiguration;
    }
});

build.initialize(gulp);

```
As you can see, we are using webpack's [DefinePlugin](https://webpack.js.org/plugins/define-plugin/) plugin to create global constants which can be configured at compile time. This way, the constant 'process.env.FARM' gets created.

On the other hand, the common TypeScript code for our site header can make the same evaluation as we've seen in the previous post:
```
var logo = require('../../common/assets/Images/logo.jpg');
if (process.env.FARM === "Standard")
{
    logo = require('../../common/assets/Images/logo.Standard.png');
}
```

There's only one thing that's still missing. We can define the value for 'FARM' when we start the SPFx solution generation. This could look like this:
```
gulp bundle --production --process.env.farm "Standard"
```

I hope you found this post useful!

In order to make this concept enterprise ready, we will need to automate the whole generation process. For this purpose, I have chosen Azure DevOps Services but I'm pretty sure the build pipeline can be similarly implemented in any other frameworks as well! Check out the details in the next post:

[Automate extensions generation for multiple SharePoint farms (Azure DevOps)](../../03/23/SharePoint-customization-for-multiple-farms-devops.html)