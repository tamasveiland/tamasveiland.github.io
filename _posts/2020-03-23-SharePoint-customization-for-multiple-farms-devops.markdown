---
layout: post
title:  "Automate extensions generation for multiple SharePoint farms (Azure DevOps)"
date:   2020-03-23 18:24:58 +0100
categories: 
    - "DevOps"
---
This is the third part of the series that I started for describing SharePoint customization creation for multiple farms. If you haven't read the first 2 posts, I recommend to do so:

[Generate extensions for multiple SharePoint farms (classic experience)](../10/SharePoint-customization-for-multiple-farms-classic.html)

[Generate extensions for multiple SharePoint farms (modern experience)](../29/SharePoint-customization-for-multiple-farms-spfx.html)

In this post, I'll elaborate on how to automate solution generation with Azure DevOps Services for multiple SharePoint farms.

Azure DevOps Services lets you define the build pipeline in YAML. You can find the more details about YAML support in the [online documentation](https://docs.microsoft.com/en-us/azure/devops/pipelines/yaml-schema?view=azure-devops&tabs=schema%2Cparameter-schema)

It already offers template for building with Node.js and Npm so this could be a good starting point. If you need some help then follow the official guidance:
[Create your first pipeline](https://docs.microsoft.com/en-us/azure/devops/pipelines/create-first-pipeline?view=azure-devops&tabs=javascript%2Cyaml%2Cbrowser%2Ctfs-2018-2#create-your-first-pipeline-1)

For the sake of simplicity, I'll only discuss the changes that are required for integrating the previously introduced concept into the build pipeline. So let's see what's needed!

First of all, we will want to specify the value for the FARM parameter in a pipeline variable. This makes it easy to change that target even for each builds. So, let's extend the list of variables as follows:
```
variables:
  system.debug: 'true'
  configuration: 'Release'
  # --process.farm valid values are:
  #     "Standard"
  #     "ExtraNet"
  #     "etc..."
  Farm: 'Standard'
  ```

Next, please recall that we wanted to build the SharePoint Framework solution the following way:
```
gulp bundle --production --process.env.farm "Standard"
```

This command can be transformed into the following pipeline task:
```
- task: Gulp@1
  displayName: 'Gulp bundle'
  inputs:
    gulpFile: 'Sources/gulpfile.js'
    targets: 'bundle'
    arguments: '--ship --process.env.farm "$(Farm)"'
    workingDirectory: 'Sources'
    gulpjs: '$(build.sourcesdirectory)\Sources\node_modules\gulp\bin\gulp.js'
    enableCodeCoverage: false
```

Actually, that will do the job for SharePoint Framework. Everything else is handled be the integrated gulp pipeline.

On the other hand, we need to achieve the same for generating the classic package. As I've written in the first post of this series, the webpack call is defined in the package.json:
```
"scripts": {
    "build-classic": "webpack --env.FARM=Standard --config classic.webpack.config.js"
    ...
  },
```

In my case, this file contains the string 'Standard' in only one place, so I can easily substitute it with the value from the pipeline variable via PowerShell script:
```
- task: PowerShell@2
  displayName: 'Replace FARM switch in package.json'
  inputs:
    targetType: 'inline'
    script: |
      [string] $token = 'Standard'
      [string] $value = '$(Farm)'
      [string] $path = '$(build.sourcesdirectory)\Sources\package.json'
      
      
      $content = Get-Content -Path $path
      $content = $content -replace $token, $value
      [System.IO.File]::WriteAllText($path, $content)
```

After that, I only have to call NPM to do the rest of the job:
```
- task: Npm@1
  displayName: 'NPM run build-classic'
  inputs:
    command: 'custom'
    workingDir: 'Sources'
    customCommand: 'run build-classic'
```

Actually, that was only a few step that we had to carry out on the template to incorporate the concept of easily building solution for multiple SharePoint farm with slightly different branding requirements.