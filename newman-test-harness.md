# Azure + Newman: A Test Harness

This workstream focussed on building tools to measure latency of the services hosted on AKS cluster. The reports created by harness would be used to compare different network configuration of the CTP infrastruture.

(FROM README:)

This is used to test the response times of a given API environment.

Request Test Harness using [Newman (from Postman)](https://www.getpostman.com/docs/v6/postman/collection_runs/command_line_integration_with_newman). This will deploy a VM and a storage account to the specified resource group.

# Getting started

Local Machine prerequisites:
 - [AzureCLI](https://docs.microsoft.com/en-us/cli/azure/install-azure-cli?view=azure-cli-latest)
 - Bash
 
Login to Azure (`az login`) and do the following steps to generate reports:

1. Modify the environment variables in the `/.env.sample` file, and configure them to your specific environment.
    - The important ones are `AZ_RESOURCE_GROUP` and `AZ_EXPIRY`. The others can be arbitrary names.
2. Save that `.env.sample` file as `.env` (which is `.gitignore`-d in the repo, so it won't be accidentally committed).
3. Run `sh deployHarness.sh`. This shell script will (with the settings provided in `.env`):
    - Create an Azure storage account and container
    - Upload the collection and environment files in `/collections` and `/environments` (see below for more info)
    - Upload the necessary report generation scripts
    - Create the test harness VM to the specified resource group
    - Execute the `newman.sh` script inside the repo to generate the reports
    - Setup a watcher to watch for files uploaded to `/collections` and `/environments` (see below for more info)
    - Echo a link to a webpage that contains links to the generated reports (in the form of, e.g., `http://123.123.123.123/reports.html`)
    
The **`/collections`** folder should contain [Postman collections](https://www.getpostman.com/docs/v6/postman/collections/creating_collections), which you can generate from Postman. The filenames don't matter, but are used in the generated report links.

The **`/environments`** folder should contain [Postman environments](https://www.getpostman.com/docs/v6/postman/environments_and_globals/manage_environments) which can also be exported from Postman. The filenames are also used in the generated report links.

The `newman.sh` script will _watch_ for those collections/environments files uploaded to the storage account. When files are uploaded, reports will be regenerated, which you can view by reloading that same link (e.g., `http://123.123.123.123/reports.html`).

The general idea is that each collection in `/collections` will contain links to the endpoints you want to test, with environment variables such as `https://{{host}}/some/api/endpoint`. Each environment file in `/environments` will be used to populate that `{{host}}` variable, and generate a report for each environment. 
