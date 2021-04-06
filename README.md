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
  "files": [
    {
      "filePath": "hello.html",
      "static": false
    },
    {
      "filePath": "cool.txt",
      "condition": "cool == 'Yes'"
    }
  ]
```

### Conditions
Conditions must follow the `<value1> <operator> <value2>` format, where `value1` and `value2` are param values or constants and `operator` can have one of the following values: `==`, `!=`, `<`, `>`, `<=`, `>=`, `&&`, or `||`.

