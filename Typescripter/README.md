# TypeScripter

## What it does
TypeScripter is a tool to generate Typescript classes from c# models.
This is super usefull for our stack (.net / angular2) because we can have typed objects from the server and we don't have to worry about our Typescript models being different than our data models.

## How to use it (see the help command)
`> TypeScripter.exe -h 

Typescripter.

    Usage:
      Typescripter.exe <SETTINGSFILE>
      Typescripter.exe <SOURCE> <DESTINATION> [<APIPATH> [ --httpclient ] [ --combineimports ]]
                       [--files=<FILES> | --class=<CLASSNAMES>]...
      Typescripter.exe ( -h | --help )

    Options:
      --files=<FILES>         Comma seperated list of .dll files to generate models from. [ default: *.client.dll ]
      --class=<CLASSNAMES>    Comma seperated list of controller class names. [ default: ApiController ]
      --httpclient            Generated data service will use the new HttpClientModule for angular 4.
      --combineimports        Combines model imports to come from the generated index file, instead of individual models. [default: false]
      -h --help               Show this screen.

      <SETTINGSFILE>          Path to a json settings file
                                   example settings file contents:
                                       {
                                            "Source": "./",
                                            "Destination": "../app/models/generated",
                                            "Files": [ "*.dll" ],
                                            "ControllerBaseClassNames": [ "ApiController" ],
                                            "ApiRelativePath": "api",
                                            "HttpModule": "HttpClientModule",
                                            "CombineImports": true|false [default: false]
                                        }
      <SOURCE>                The path that contains the .dll(s)
      <DESTINATION>           The destination path where the generated models will be placed
      <APIPATH>               The prefix api calls use (leave blank to not generate a data service)

## Post-build Event
The easiest way to utilize Typescripter is via a post-build event in the client project. Here's an example:
```
for /f %%t in ('dir $(SolutionDir)packages /B /S ^| find "TypeScripter.exe"') do %%t ./typescripter.json
```
 For each TypeScripter executable in the packages directory, execute it with the given json configuration. There is a side effect of this configuration, however, because if you have multiple versions of TypeScripter in your solution's package directory, it will generate models for each version.