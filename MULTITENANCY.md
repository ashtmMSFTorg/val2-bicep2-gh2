# Multi-tenant Deployments

## Setup

Run `symphony provision` as you normally would.

Provision for TF? Yes.

Create resources? Yes.  (storage account for state files)

Create SP? No.

You'll be prompted for details of your own SP: enter the subscriptionId (currently being used by AZ CLI), the tenantId in which you registered the SP, and the resulting applicationId.

(Note: seems to be a bug with Symphony where it saves an incorrect SP name in symphony.json if you BYO SP)

### Home tenant
Deployments can only be made across tenant boundaries with an application which has been registered with a multi-tenant audience.

Create a new app registration in Entra ID; provide any name you like, and when asked about the audience or supported account types, select "Accounts in any organizational directory (Any Microsoft Entra ID tenant - Multitenant)".

```bash
application_id=$(az ad app create --display-name "symphony-conductor" --sign-in-audience AzureADMultipleOrgs | jq -r '.appId')
echo "Application Id = $application_id"
```

Make a note of the applicationId (== clientId); you'll need this when configuring any tenants targeted for deployments.

### Destination tenant
Before the application has permission to do anything in the target tenant, you must first provision a service principal for it there and grant it the required access.

You can do this by logging into the target tenant with the AZ CLI and running the following bash script:

```bash
application_id="__REPLACE_ME__"
subscription_id="__REPLACE_ME__"

sp_oid=$(az ad sp create --id $application_id | jq -r '.id')

az role assignment create --assignee-object-id $sp_oid --assignee-principal-type ServicePrincipal --role "Owner" --scope "/subscriptions/$subscription_id"
```

If you need an environment in which to quickly run this, consider opening a Cloud Shell (bash) in the Azure Portal while logged into the tenant.

## MT for TF

Once you provision a new Terraform-based repo, you can get started with cross tenant deployments by creating a new environment and populating the `target_tenant_id` and `target_subscription_id` variables.

For example, copy the entire prod environment folder to a new environment named external.

Then update all the *.tfvars.json files in that environment. Change the value of the existing `env` to `external`, then add `target_tenant_id` and `target_subscription_id` variables with appropriate values for the tenant + subscription combination you'd like this environment to represent.

## MT for Bicep
