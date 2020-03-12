# ARM Template of Invincible Infrastructure Hack.

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fedm-ms%2Finvincible-env%2Fmaster%2FarmTemplates%2Ftemplate.json" target="_blank" rel="noopener noreferrer">


<img src="http://azuredeploy.net/deploybutton.png"/>

</a>

<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fedm-ms%2Finvincible-env%2Fmaster%2FarmTemplates%2Ftemplate.json" target="_blank" rel="noopener noreferrer">

<img src="http://armviz.io/visualizebutton.png"/>

</a>

This is a modified template that started as an export from the Azure Portal. The [exported template](https://github.com/edm-ms/invincible-env/blob/master/armTemplates/Portal%20Export/template.json) is 620 lines long, and [this template](https://github.com/edm-ms/invincible-env/blob/master/armTemplates/template.json) is 336 lines long!

How did we do this?

[Copy Loops](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/copy-variables#variable-iteration)

```json
  "type": "Microsoft.Web/serverfarms",
  "apiVersion": "2018-02-01",
  "name": "[concat(variables('appName'), '-', variables('appLocations')[copyIndex()], variables('appServiceSuffix'))]",
  "copy": {
      "count": "[length(variables('appLocations'))]",
      "name": "appLoop"
  },
  "location": "[variables('appLocations')[copyIndex()]]",
```

We also use copy loops inside of a resource property to itterate over properties of a resource instead of the resource itself.

```json
"name": "DefaultBackendPool",
"properties": {
    "copy": [
        {
            "name": "backends",
            "count": "[length(variables('appLocations'))]",
            "input": {
                "address": "[concat(variables('appName'), '-', variables('appLocations')[copyIndex('backends')], '-web.azurewebsites.net')]",
                "httpPort": 80,
                "httpsPort": 443,
                "priority": 1,
                "weight": 50,
                "backendHostHeader": "[concat(variables('appName'), '-', variables('appLocations')[copyIndex('backends')], '-web.azurewebsites.net')]",
                "enabledState": "Enabled"
            }
        }
    ]
```

We use variables to define the names of resources, and the entire deployment is modified by just setting where we want to deploy.
```json
"appLocations": [
    "eastus",
    "centralus",
    "westcentralus",
    "westus2"
]
```

We also use the template function [uniqueString](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-string#uniquestring) to help construct a unique but repeatable name for the app.
Passing in the appName parameter, resource group name, and subscription ID to generate our hash to add to the resources.
We leverage [take](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-array#take) to grab 6 characters for this string. If we deploy in the same sub, resource group, and use the same
appName we will end up with the same hash. Using this in a different sub, resource group, or app name will generate a 
different string.

```json
"randomAppName": "[take(uniqueString(parameters('appName'), resourceGroup().name, subscription().subscriptionId), 6)]",
```