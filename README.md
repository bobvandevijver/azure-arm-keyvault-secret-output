# Azure ARM Keyvault helper
Azure ARM template created to output the inputted keyvault secret back to the consumer.

By default, Azure can only use [dynamic Keyvault secrets](https://docs.microsoft.com/nl-nl/azure/azure-resource-manager/resource-manager-keyvault-parameter#reference-secrets-with-dynamic-id) in linked templates, which are not convenient to use as you will need to deploy them to a storage which can be accessed by Azure during the deployment. 

This repository helps you by defining a simple template, which has the sole purpose to return the inserted secret back to you original template. You do not need to worry about the availability of this template, as it is freely available on Github.

## Usage

Your template should reference this template, and set the `inputSecret` as keyvault reference. You can then use it in your template by referencing the `outputSecret`. It's that simple!

Example (you're free to use variables and parameters wherever you want):
```json
{
  "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "resources": [
    {
      "apiVersion": "2018-05-01",
      "name": "password-resource",
      "type": "Microsoft.Resources/deployments",
      "properties": {
        "mode": "Incremental",
        "templateLink": {
          "contentVersion": "1.0.0.0",
          "uri": "https://raw.githubusercontent.com/bobvandevijver/azure-arm-keyvault-secret-output/v1.0.0/template.json"
        },
        "parameters": {
          "inputSecret": {
            "reference": {
              "keyVault": {
                "id": "[resourceId('Microsoft.KeyVault/vaults', 'keyvault')]"
              },
              "secretName": "sqlserverpassword"
            }
          }
        }
      }
    },
    {
      "comments": "DbManagement SQL Server",
      "name": "sqlserver",
      "type": "Microsoft.Sql/servers",
      "apiVersion": "2014-04-01",
      "location": "[resourceGroup().location]",
      "properties": {
        "version": "12.0",
        "administratorLogin": "admin",
        "administratorLoginPassword": "[reference('password-resource').outputs.outputSecret.value]"
      },
      "dependsOn": ["password-resource"]
    }
  ]
}
```
