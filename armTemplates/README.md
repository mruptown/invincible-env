# Invincible Infrastructure Hack (ARM).

## Contents
- [Invincible Infrastructure Hack (ARM).](#invincible-infrastructure-hack-arm)
  - [Contents](#contents)
    - [What is this?](#what-is-this)
    - [Copy Loops](#copy-loops)
    - [Variables](#variables)
    - [Functions](#functions)
    - [ARM Deployments](#arm-deployments)
    - [What next?](#what-next)

---

### What is this?

 This is an ARM template deployment of the "invincible infrastructure" hack that was built using AZ CLI. The goal here is to show an alternative method of deployment using ARM templates, and the benefits of a declarative language. This is a modified template that started as an export from the Azure Portal. The [exported template](https://github.com/edm-ms/invincible-env/blob/master/armTemplates/Portal%20Export/template.json) is 620 lines long, and [this template](https://github.com/edm-ms/invincible-env/blob/master/armTemplates/template.json) is only 336 lines long!

 Using the "Deploy to Azure" button below you can deploy this into your Azure subscription. The "Visualize" button will show you the resources being deployed, but it does not understand copy loops (more below) so the quantity of resources will not be displayed.

 <a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fedm-ms%2Finvincible-env%2Fmaster%2FarmTemplates%2Ftemplate.json" target="_blank" rel="noopener noreferrer">


<img src="http://azuredeploy.net/deploybutton.png"/>

</a>

<a href="http://armviz.io/#/?load=https%3A%2F%2Fraw.githubusercontent.com%2Fedm-ms%2Finvincible-env%2Fmaster%2FarmTemplates%2Ftemplate.json" target="_blank" rel="noopener noreferrer">

<img src="http://armviz.io/visualizebutton.png"/>

</a>

### Copy Loops

We leverage [copy loops](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/copy-variables#variable-iteration) so we don't need to define the same resource over and over again in the template. Doing this greatly reduces the size of the overall template.

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

### Variables 

We use variables to define the names of resources.

```json
"cosmosName": "[concat(parameters('appName'), '-', variables('randomAppName'), '-cosmos')]",
"dbName": "[concat(parameters('appName'), '-db')]",
"frontDoorName": "[concat(parameters('appName'), '-', variables('randomAppName'), '-afd')]",
"appName": "[concat(parameters('appName'), '-', variables('randomAppName'))]",
```

With the use of variable names creating the resources the entire deployment is modified by just setting where we want to deploy. The copy loop we use loops through the length of this array, and names resources with the region they are deployed.

```json
"appLocations": [
    "eastus",
    "centralus",
    "westcentralus",
    "westus2"
]
```

### Functions

We use the template function [uniqueString](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-string#uniquestring) to help construct a unique but repeatable name for the app. Passing in the appName parameter, resource group name, and subscription ID to generate our hash to add to the resources. We leverage [take](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/template-functions-array#take) to grab 6 characters for this string. If we deploy in the same sub, resource group, and use the same appName we will end up with the same hash. Using this in a different sub, resource group, or app name will generate a different string.

```json
"randomAppName": "[take(uniqueString(parameters('appName'), resourceGroup().name, subscription().subscriptionId), 6)]",
```

### ARM Deployments 

When we push our template to the Azure Resource Manager (ARM) we are able to track all elements of the deployment.

<img src="/armTemplates/images/deployment.png">

We can drill into the deployment details and see information about each piece of the deployment.

<img src="/armTemplates/images/deploymentInfo.png">

### What next? 
1. What other thing could we change from being hard-coded to being passed as a parameter?
2. How could we use conditional deployments to deploy per-environment?
3. Could we deconstruct this larger template into smaller templates?
4. What would happen if we deployed the template in ["complete"](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes#complete-mode) vs. ["incremental"](https://docs.microsoft.com/en-us/azure/azure-resource-manager/templates/deployment-modes#incremental-mode)?