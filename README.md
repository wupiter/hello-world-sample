This is a sample template git repo for [Wupiter](https://wupiter.com), a web app that allows dev teams to easily create custom code generators.

These code generators (Accelerators) can help reduce repetitive coding tasks (ie: creating new projects), share best practices and accelerate ramp up time on new projects.

# Overview
Each Accelerator (template repo) is required to have a `wupiter.json` file that includes metadata information:
- basic details on the accelerator (name, description, tags, etc.)
- input params that users can use to customize the code generator
- template files that are used to generate the output (a zip file that can be downloaded from Wupiter)

Besides the `wupiter.json` file, the git repo also needs to include the files needed for the code generation (referenced within the `wupiter.json` file).

## Basic details
These information help users search Accelerators and understand what these Accelerators do.

Currently supported fields: 
- name
- description
- tags

Sample:
```
...
  "name": "Hello World",
  "description": "Sample code generator",
  "tags": [ "html" ],
...  
```

## Params
In Wupiter there's an Accelerator Generate page where users can run the code generator. On this page users can customize the behavior by setting the Accelerator params' values before running the code generator.

### Common param fields
These fields are supported for all params:
- name (required): Used in template files as variables
- label (optional): Human friendly label of the web UI component. If not set, the UI will display the capitalized name as label
- type (required): Defines the UI component type to use on the Accelerator's Generate page
- description (optional): Provides guidance on the param's usage
- required (optional): If `true`, user needs to set this param's value (default is `false`)

### Text param
These params have a `text` type and are displayed as an input text field.

Supported fields:
- defaultValue (optional): Used as the default value of the param

Sample:
```
    {
      "name": "name",
      "type": "text",
      "description": "Enter your name",
      "defaultValue": "World",
      "required": true
    }
```

### Checkbox param
These params have a `checkbox` type and are displayed as a checkbox.

Supported fields:
- defaultValue (optional): Used as the default value of the param. If it's `true`, the checkbox will be checked

Sample:
```
    {
      "name": "checked",
      "type": "checkbox",
      "description": "Check me",
      "defaultValue": true,
      "required": true
    }
```

### Radios param
These params have a `radios` type and are displayed as radio buttons.

Supported fields:
- defaultValue (optional): Used to select the default radio button to be checked
- values (required): Array of strings (ie: `[ "Yes", "No" ]`) or objects (ie: `[ { "label": "Yes", "key": "yes" }, { "label": "No", "key": "no" } ]`, where `label` is the displayed label and `key` is the value set to the param, if the user selects the radio button)

Sample:
```
    {
      "name": "cool",
      "type": "radios",
      "label": "Are you cool?",
      "defaultValue": "Yes",
      "values": [ "Yes", { "label": "No", "key": "no" } ],
      "required": false
    }
```

### Select param
These params have a `select` type and are displayed as a select form element.

Supported fields:
- defaultValue (optional): The option item with this value will be selected
- values (required): Array of strings (ie: `[ "Yes", "No" ]`) or objects (ie: `[ { "label": "Yes", "key": "yes" }, { "label": "No", "key": "no" } ]`, where `label` is the displayed label and `key` is the value set to the param, if the user selects the option item)

Sample:
```
    {
      "name": "color",
      "type": "select",
      "label": "Choose a color",
      "defaultValue": "Red",
      "values": [ "Red", { "label": "Blue", "key": "blue" }, "Green" ],
      "required": false
    }
```

### Searchbox param
These params have a `searchbox` type and are displayed as a searchable dropdown with selected values displayed as tags.

Supported fields:
- allowMultiple (optional): If `true`, user can select multiple values, otherwise only a single value can be selected
- values (required): Array of objects, see sample below. Within these objects, label field is what gets displayed and key is what gets stored (and passed to template files)

Sample with simple values:
```
    {
      "name": "birthPlace",
      "type": "searchbox",
      "label": "Where were you born?",
      "required": false,
      "allowMultiple": false,
      "values": [
        { "label": "Canada", "key": "CA" },
        { "label": "Ireland", "key": "IE" },
        { "label": "United States of America", "key": "US" }
      ]
    }
```

Sample with groupped values:
```
    {
      "name": "cities",
      "type": "searchbox",
      "label": "Favourite cities?",
      "required": false,
      "allowMultiple": true,
      "values": [
        {
          "groupLabel": "Canada",
          "values": [
            { "label": "Toronto" },
            { "label": "Ottawa" },
            { "label": "Montreal" }
          ]
        },
        {
          "groupLabel": "US",
          "values": [
            { "label": "New York" },
            { "label": "Washington DC" },
            { "label": "Philadelphia" }
          ]
        }
      ]
    }
```
*In the searchbox dropdown there'll be an item for the group and for each value within the group. When user selects a group in the UI, all its values will be selected.*

## Pre-processing
This field allows configuring string replacement operations on both file paths and file contents before the template files are processed (see next section on templates).

Sample:
```
  "preProcessing": {
    "pathReplacements": [
      { "target":  "demo-api", "replacement":  "{{appName}}"}
    ],
    "textReplacements": [
      { "target":  "demo-api", "replacement":  "{{appName}}"},
      { "target":  "#!#", "replacement":  ""},
      { "target":  "<!--#", "replacement":  ""},
      { "target":  "#-->", "replacement":  ""},
      { "target":  "//#", "replacement":  ""}
    ]
  },
```

In above example
- `demo-api` will be replaced in both file paths and file content with the value of the `appName` param
- comments like `#!#{{appName}}` (ie: in YAML files) will be replaced with `{{appName}}`
- comments like `<!--#{{appName}}#-->` (ie: in XML files) will be replaced with `{{appName}}`
- comments like `//#{{appName}}` (ie: in XML files) will be replaced with `{{appName}}`

Using comments in the code that will later be turned to actual code can be useful to hide a logic that could otherwise make the code not compile or fail to parse.

By default, Wupiter also replaces `//{{`, `#{{` and `<!--{{` to `{{` and `}}-->` to `}}` in all files (aka text replacements), so we can embed template logic in files without breaking compilation or parsing. So no need to include those text replacements.

## Template files
The files defined in the `wupiter.json` file under the `files` field will be included in the output zip file created by the code generator. 

Currently, [HandlebarsJS](https://handlebarsjs.com/) is used as the template engine and the input param values are passed in as template variables to these templates.

Sample template file:
```
<html>
<body>
  Hello, {{name}}!
</body>
</html>
```
*Where `name` is one of the params' name.*

Supported fields for files:
- filePath (required): File path within the git repo and the generated zip file. Use `/` as folder separator
- static (optional): Code generator will copy these files to the output zip file as-is. Default is `false`
- condition (optional): if `condition` field is provided, file will only be included in the output, if the condition evaluates to `true`

Sample `files` section within `wupiter.json`:
```
  "files": {
    "hello.html": {
      "static": false
    },
    "cool.txt": {
      "condition": "cool == 'Yes'"
    }
  ]
```

### Conditions
Conditions must follow the `<value1> <operator> <value2>` format, where `value1` and `value2` are param values or constants and `operator` can have one of the following values: `==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, or `||`.

### Supported Handlebar helpers

#### capitalize
Changes the first letter to upper case of input string.

Sample:
```
{{capitalize name}}
```

If `name=john`, it will output `John`.

#### CamelCase
Applies CamelCase format on input string:
- it will change the first letter to upper case
- it will change the next letter after a whitspace to upper case and remove the whitespace

Sample:
```
{{CamelCase appName}}
```

If `appName=todo api`, it will output `TodoApi`.

#### strReplace
Replaces all occurences of a pattern in a string with another string.

Sample:
```
{{strReplace packageName '.' '/'}}
```

If `packageName=com.example.app`, it will output `com/example/app`.

#### eval
It will evaluate an operation on two expressions. Supported operations: ==, !=, <, <=, >=, >, && and ||.

Sample:
```
<!--{{#if (eval javaVersion '==' 11)}}-->
<java.version>11</java.version>
<!--{{/if}}-->
<!--{{#if (eval javaVersion '!=' 11)}}-->
<!--#<java.version>{{javaVersion}}</java.version>#-->
<!--{{/if}}-->
```

It will output `<java.version>JAVA_VERSION</java.version>` where `JAVA_VERSION` is the value passed to the `javaVersion` param.

Please, note the `#` before the opening `if` element.

