---
layout: post
title:  "Generate extensions for multiple SharePoint farms (classic experience)"
date:   2020-02-10 19:23:41 +0100
categories: 
    - "SharePoint"
---
Have you ever found yourself in a situation that you had to support that same SharePoint customization on different farms with slightly different content? I did! At first sight, this isn't the easiest job you would ask for yourself.
After some research, it turned out, it isn't the worst, at all.

In this post, I'm going to provide you the details how to differentiate between SharePoint farms with just a simple Webpack configuration step both for classic and modern experience!

Let's say, we have the requirement to show different logo in the site header for different farms. So we need branding customization for our farms. For the sake of maintainability, it's a best practice to use as much common code as possible for the classic and modern experience. Fortunately, there's a sample on github which showcase how to easily solve this:
[React Menu Footer Classic Modern](https://github.com/SharePoint/sp-dev-fx-extensions/tree/master/samples/react-menu-footer-classic-modern "React sample with common code")

Now, let's dive into the details!

# Classic Experience
If you'd like to generate a ScriptLink CustomAction for the classic experience then you'll probably use webpack to generate the javascript file. In this case, you can define a new script for building you bundle in the package.json as follows:
```
"scripts": {
    "build-classic": "webpack --env.FARM=Standard --config classic.webpack.config.js"
    ...
  },
```
We'll talk about the FARM environment variable a little bit later but we need some more details to understand how this can work. That being said, let's take a look into the above mentioned 'classic.webpack.config.js'. Here you can find a small snippet that is responsible for propagating environment variables to webpack:
```
const webpack = require('webpack');
const path = require('path');

module.exports = env => {

    const envKeys = Object.keys(env).reduce((prev, next) => {
        prev[`process.env.${next}`] = JSON.stringify(env[next]);
        return prev;
      }, {});

    return {
        ...
        plugins: [
            // add the plugin to your plugins array
            new webpack.DefinePlugin(envKeys)
        ]
        ...
    };
} 
```
Finally, you can make your appropriate if-else conditional statements in the source code. In our case, it would mean loading different images for the site header:
```
var logo = require('../../common/assets/Images/logo.jpg');
if (process.env.FARM === "Standard")
{
    logo = require('../../common/assets/Images/logo.Standard.png');
}
```
let's sum up what we've seen. If you need to load different images for different SharePoint farms, all you need to do is:
1. Extend your webpack config (propagate input variables to webpack),
2. Insert conditional loading rules into the TypeScript code, and
3. Update the value of the input variable in the packages.json depending on target farm you are building for.

That's it, pretty simple.
What about the modern experience? Check out my next post:
[Generate extensions for multiple SharePoint farms (modern experience)](../29/SharePoint-customization-for-multiple-farms-spfx.html)

