---
title: "Key Vault"
date: 2020-02-05T08:53:46+01:00
chapter: false
weight: 11
---

### Overview
The KeyVault package contains *four* builders, for the different components used by KeyVault: One for **access policies**, one for **secrets**, one for the overall **keyvault** container, and one for **adding access policies** to an existing key vault.

* Keys (`Microsoft.KeyVault/vaults/keys`)
* KeyVault (`Microsoft.KeyVault/vaults`)
* Secrets (`Microsoft.KeyVault/vaults/secrets`)
* AccessPolicies (`Microsoft.KeyVault/vaults/accessPolicies`)

#### Key Builder
The `key` builder allows you to generate RSA and elliptical curve keys in the key vault.

| Keyword | Purpose |
|-|-|
| name | Sets the name of the key to generate. |
| key_operations | Sets the operations that they generated key can be used to perform. |
| key_type | Sets the type of key. Helpers are defined for many typical types: RSA_2048, RSA_3072, RSA_4096, EC_P256, EC_P384, EC_P521, EC_P256K |
| status | Enables or disables the key (defaults to 'Enabled'). |
| activation_date | Sets the activation date of the key. |
| expiration_date | Sets the expiration date of the key. |
| link_to_unmanaged_keyvault | Links this key to an existing keyvault and allows the key to be deployed standalone |
| depends_on | [Sets the dependencies of the key vault.](../../dependencies/) |
| add_tag | Adds a tag to the secret. |
| add_tags | Adds multiple tags to the secret. |

#### Secret Builder
The `secret` builder allows you to store secrets into key vault. Values for a secret are passed by Secure String parameters.

| Keyword | Purpose |
|-|-|
| name | Sets the name of the secret. |
| value | Sets the name of the secure string parameter that will contain the value of the secret. |
| content_type | Sets the content type of the secret. |
| status | Enables or disables the secret (defaults to 'Enabled'). |
| activation_date | Sets the activation date of the secret. |
| expiration_date | Sets the expiration date of the secret. |
| link_to_unmanaged_keyvault | Links this secret to an existing keyvault and allows the secret to be deployed standalone |
| depends_on | [Sets the dependencies of the key vault.](../../dependencies/) |
| add_tag | Adds a tag to the secret. |
| add_tags | Adds multiple tags to the secret. |

#### Access Policy Builder
The `accessPolicy` builder allows you to create access policies for key vault.

| Keyword | Purpose |
|-|-|
| object_id | Sets the Object ID of the permission set. |
| application_id | Sets the Application ID of the permission set. |
| key_permissions | Sets the Key permissions of the permission set. |
| storage_permissions | Sets the Storage permissions of the permission set. |
| secret_permissions | Sets the Secret permissions of the permission set. |
| certificate_permissions | Sets the Certificate permissions of the permission set. |

#### Key Vault Builder
The `keyVault` builder contains access policies, secrets, and configuration information to create a full key vault account.

| Keyword | Purpose |
|-|-|
| name | Sets the name of the vault. |
| sku | Sets the sku of the vault. |
| tenant_id | Sets the Tenant ID of the vault. |
| enable_vm_access | Allows VM access to the vault. |
| disable_vm_access | Disallows VM access to the vault. |
| enable_resource_manager_access | Allows Resource Manager access to the vault. |
| disable_resource_manager_access | Disallows Resource Manager access to the vault. |
| enable_disk_encryption_access | Allows Azure Disk Encyption service access to the vault. |
| disable_disk_encryption_access | Disallows Azure Disk Encyption service access to the vault. |
| enable_rbac | Enables Azure role based access control for data access.
| disable_rbac | Disables Azure role based access control for data access.
| enable_soft_delete | Enables VM access to the vault. |
| enable_soft_delete_with_purge_protection | Disables VM access to the vault. |
| uri | Sets the URI of the vault. |
| enable_recovery_mode | Sets the Creation Mode to Recovery. |
| disable_recovery_mode | Sets the Creation Mode to Default. |
| add_access_policy | Adds an access policy to the vault. |
| add_access_policies | Adds access policies to the vault. |
| enable_azure_services_bypass | Allows Azure traffic can bypass network rules. |
| disable_azure_services_bypass | Disallows Azure traffic can bypass network rules. |
| allow_default_traffic | Allow traffic if no rule from ipRules and virtualNetworkRules match. This is only used after the bypass property has been evaluated. |
| deny_default_traffic | Deny traffic when no rule from ipRules and virtualNetworkRules match. This is only used after the bypass property has been evaluated. |
| add_ip_rule | Adds an IP address rule. This can be an IPv4 address range in CIDR notation, such as '124.56.78.91' (simple IP address) or '124.56.78.0/24' (all addresses that start with 124.56.78). |
| add_vnet_rule | Adds a virtual network rule. This is the full resource id of a vnet subnet, such as '/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/test-vnet/subnets/subnet1'. |
| add_secret | Adds a secret to the vault. This can either be a "full" secret config created using the Secret Builder, a string literal value which represents the parameter name, or a string literal with a resource and an expression based on that resource e.g. a storage account and the Key member. |
| add_secrets | Adds multiple secrets to the vault. This can either be "full" secret configs created using the Secret Builder, string literal values which represents the parameter name. |
| add_tag | Adds a tag to the key vault. |
| add_tags | Adds multiple tags to the key vault. |

#### Key Vault Add Access Policy Builder (`keyVaultAddPolicies`)
As applications grow, more components often need access to a key vault. The `keyVaultAddPolicies` builder is used to add new access policies to an existing key vault without the need to redeploy the full key vault, potentially changing existing access.

| Keyword | Purpose |
|-|-|
| key_vault | Sets the resource Id or a builder for the existing key vault where the policies should be added. |
| add_access_policies | A list of policies to add to the key vault. |
| tenant_id | Used if granting access to users or service principals from another tenant. |

#### Configuration Members

| Member | Purpose |
|-|-|
| VaultUri | Gets the ARM expression path to the key vault's URI. |

#### Utilities
* The KeyVault module comes with a set of utility functions to quickly create access policies if you do not wish to use the `AccessPolicy` builder, in the `Farmer.KeyVault.AccessPolicy` module which enable creating an access policy for a `PrincipalId` or an `ObjectId` which will have the GET Secret permission.
* In addition, the `AccessPolicy` module also contains helpers to search for users or groups in active directory (*requires Azure CLI installed*), as well as their Object IDs. These can be used to rapidly create Access Policies for specific users.

#### Example

```fsharp
open Farmer
open Farmer.Builders
open System

let policy =
    accessPolicy {
        object_id Guid.Empty
        application_id Guid.Empty
        certificate_permissions [ KeyVault.Certificate.List ]
        secret_permissions KeyVault.Secret.All
        key_permissions [ KeyVault.Key.List ]
    }

let complexSecret = secret {
    name "myComplexSecret"
    content_type "application/text"
    status Enabled
    activation_date (DateTime.Today.AddDays -1.)
    expiration_date (DateTime.Today.AddDays 1.)
}

let vault =
    keyVault {
        name "MyVault"
        sku KeyVault.Sku.Standard
        tenant_id Guid.Empty

        enable_disk_encryption_access
        enable_resource_manager_access
        enable_soft_delete_with_purge_protection

        disable_vm_access
        enable_recovery_mode
        add_access_policy policy
        enable_azure_services_bypass

        add_ip_rule "127.0.0.1"
        add_vnet_rule "/subscriptions/subid/resourceGroups/rg1/providers/Microsoft.Network/virtualNetworks/test-vnet/subnets/subnet1"
        allow_default_traffic

        add_secret complexSecret
        add_secret "simpleSecret"
        add_secrets [ "firstSecret"; "secondSecret"]
        add_keys [
            key {
                name "myRsaKey"
                key_type KeyType.RSA_4096
            }
            key {
                name "myRllipticalCurveKey"
                key_type KeyType.EC_P256
            }
        ]
    }

arm {
    add_resource vault
    output "vault-uri" vault.VaultUri
}

```