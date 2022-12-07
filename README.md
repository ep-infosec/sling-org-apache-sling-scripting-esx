[![Apache Sling](https://sling.apache.org/res/logos/sling.png)](https://sling.apache.org)

&#32;[![Build Status](https://ci-builds.apache.org/job/Sling/job/modules/job/sling-org-apache-sling-scripting-esx/job/master/badge/icon)](https://ci-builds.apache.org/job/Sling/job/modules/job/sling-org-apache-sling-scripting-esx/job/master/)&#32;[![Coverage](https://sonarcloud.io/api/project_badges/measure?project=apache_sling-org-apache-sling-scripting-esx&metric=coverage)](https://sonarcloud.io/dashboard?id=apache_sling-org-apache-sling-scripting-esx)&#32;[![Sonarcloud Status](https://sonarcloud.io/api/project_badges/measure?project=apache_sling-org-apache-sling-scripting-esx&metric=alert_status)](https://sonarcloud.io/dashboard?id=apache_sling-org-apache-sling-scripting-esx)&#32;[![Contrib](https://sling.apache.org/badges/status-contrib.svg)](https://github.com/apache/sling-aggregator/blob/master/docs/status/contrib.md)&#32;[![scripting](https://sling.apache.org/badges/group-scripting.svg)](https://github.com/apache/sling-aggregator/blob/master/docs/groups/scripting.md) [![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://www.apache.org/licenses/LICENSE-2.0)

# Apache Sling Scripting ESX

This module is part of the [Apache Sling](https://sling.apache.org) project.

A Node JS (like) module loader for Apache Sling.

This module is considered **experimental** for now.

## Description
This module implements a Nashorn Apache Sling Script Engine for the "esx" extension.

It requires a function named `render` in the `esx` script that processes the 
request.

## Installation
The `org.apache.sling.fragment.nashorn` bundle must be installed before this bundle, to export the `jdk.nashorn.api.scripting` package.

Currently this implementation requires **java version "1.8.0_92"** or higher

## Usage
Once this bundle is active you can try the engine with this minimal (and not very interesting) example:

First create a node with some content:

    curl -u admin:admin \
      -F"sling:resourceType=foo" \
	  -Ftitle="Hello ESX" \
	  -Ftext="Here's some example text" \
	  http://localhost:8080/apps/foo

Then create an ESX script to render it:

    $ cat << EOF > /tmp/foo.esx
    var foo = {
      render: function () {
        var output  = \`<h1>\${currentNode.properties.title}</h1>\`;
        output += currentNode.properties.text;
        return output;     
      }
    }  
    module.exports = foo;
    EOF

    $ curl -u admin:admin -T /tmp/foo.esx http://localhost:8080/apps/foo/foo.esx

    $ curl http://localhost:8080/apps/foo.html
    <h1>Hello ESX</h1>Here's some example text


An ESX file is a regular java script file.

The NodeJS module resolution (https://nodejs.org/api/modules.html) is implemented to give access to the
rich collection of Node modules.

There's currently no priority handling of global modules.

The engine searches for scripts in the following order, if the regular module resolution does not find a module:
        - /apps/esx/node_modules
        - /apps/esx/esx_modules
        - /libs/esx/node_modules
        - /libs/esx/esx_modules

Additionally, ESX will try to resolve the folder *esx_modules* prior to *node_modules*.

### Special Loaders
Require Extensions are deprecated (see https://nodejs.org/api/globals.html#globals_require_extensions), therefore we have not implemented/used the extension loaders api and .bin extension cannot be used.

We have borrowed the requirejs loader plugin syntax instead (see http://requirejs.org/docs/api.html#text). Additionally to the standard JS loader following two loaders are existing:

- text (e.g. ```require("text!./templates/header.html"))```)
  - will return a javascript native string containing the content of the file
- resource  (e.g. ```require("resource!./content/blogposts)```)
  following will be exposed:
  - properties (resource valuemap)
  - path (jcr path)  
  - simpleResource (has getChildren method with resolved simpleresoruce in an array)
  - array with list of children (simpleResource)

- json loader  (e.g. ```require("./dict/en.json```)
  - the json as a whole will be exported as a javascript Object

##  Demo Application
Currently the demo application is bundles with the engine bundle.

open http://localhost:8080/libs/esx/demo/content/demo.html

### Writing a module
You can actually follow the NODE JS description on https://nodejs.org/api/modules.html for more detailed explanation.

A module has access to following variables:
- __filename
- __dirname
- console (console.log is a log4j logger registered to the resolved module path and is not a 1:1 console.log implementation for now)
- properties (valuemap)
- simpleResource
- currentNode
 - currentNode.path
 - currentNode.resource
 - currentNode.properties
- sling (SlingScriptHelper)


# Example
## Caluclator Module
Path: /apps/demo/components/test/helper/calculator/index.js
```javascript
function calculate(a, b) {
  return a + b;
}
exports.math = calculate;
```

## Test components
Path: /apps/demo/components/test/test.esx
```javascript
var calculator = require("./helper/calculator");

exports.render = function () {
  return calculator.math(2,2);
}
```
