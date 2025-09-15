# V. Azure Vault

## 1. Lil' intro ?

Azure Key Vault permet de **stocker des secrets** (mots de passe, clés, tokens, etc.) de manière centralisée, sécurisée et contrôlée. On l'utilise ici pour partager un secret généré aléatoirement entre notre machine locale et notre VM Azure, de façon sécurisée.

Cela évite d'écrire des secrets en clair dans le code Terraform ou ailleurs.

---

## 2. Do it !

###  Fichiers utilisés

- `main.tf`
- `keyvault.tf`
- `variables.tf`
- `outputs.tf`

###  Fichier `variables.tf`

```hcl
variable "keyvault_name" {
  default = "vaultcauchemar"
}

variable "keyvault_secret_name" {
  default = "cauchemar-secret"
}
```

###  Fichier `keyvault.tf`

```hcl
data "azurerm_client_config" "current" {}

resource "random_password" "vault_secret" {
  length           = 16
  special          = true
  override_special = "@#$%&*()-_+="
}

resource "azurerm_key_vault" "cauchemar_vault" {
  name                       = var.keyvault_name
  location                   = azurerm_resource_group.main.location
  resource_group_name        = azurerm_resource_group.main.name
  tenant_id                  = data.azurerm_client_config.current.tenant_id
  sku_name                   = "standard"
  soft_delete_retention_days = 7
  purge_protection_enabled   = false

  access_policy {
    tenant_id = data.azurerm_client_config.current.tenant_id
    object_id = data.azurerm_client_config.current.object_id

    secret_permissions = [
      "Get", "List", "Set", "Delete", "Purge", "Recover"
    ]
  }

  access_policy {
    tenant_id = data.azurerm_client_config.current.tenant_id
    object_id = azurerm_linux_virtual_machine.main.identity[0].principal_id

    secret_permissions = [
      "Get", "List"
    ]
  }
}

resource "azurerm_key_vault_secret" "vault_secret" {
  name         = var.keyvault_secret_name
  value        = random_password.vault_secret.result
  key_vault_id = azurerm_key_vault.cauchemar_vault.id
}
```

###  Fichier `outputs.tf`

```hcl
output "vault_secret" {
  value     = azurerm_key_vault_secret.vault_secret.value
  sensitive = true
}
```

###  Déploiement

```bash
terraform apply
```

Résultat :
```bash
azurerm_key_vault.cauchemar_vault: Creation complete after 2m41s
azurerm_key_vault_secret.vault_secret: Creation complete
```

---

## 3. Proof proof proof

###  Depuis la machine locale

```bash
az keyvault secret show --name "cauchemar-secret" --vault-name "vaultcauchemar" --query "value" -o tsv
```

 **Sortie (masquée ici pour la confidentialité)**

```txt
[secret-string-masqué]
```

###  Depuis la VM Azure

Script utilisé pour accéder au secret via `curl` (Azure Instance Metadata Service + access token) :

```bash
#!/bin/bash

TOKEN=$(curl -s --request GET   --url 'http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://vault.azure.net'   --header "Metadata:true" | jq -r '.access_token')

curl -s https://vaultcauchemar.vault.azure.net/secrets/cauchemar-secret?api-version=7.0   --header "Authorization: Bearer $TOKEN" | jq -r '.value'
```

 Le script retourne bien le contenu du secret. Cela confirme que **la VM a bien les droits d'accès**.

---

##  Comprendre ce qui se passe (IMDS + JWT)

- Azure met à disposition un service de métadonnées interne (IMDS) à `169.254.169.254`.
- Lorsqu’on envoie une requête avec le header `Metadata:true`, on obtient un **jeton d’accès (JWT)** signé par Azure.
- Ce JWT peut ensuite être utilisé dans une requête HTTP classique (avec `curl`) vers l’API d’Azure Key Vault.
- Cela permet à la VM de s’authentifier sans login/mot de passe.

---

##  Résumé validé

-  Terraform a bien créé la Key Vault et le secret aléatoire.
-  Le secret est accessible depuis la machine locale avec `az keyvault secret show`
-  Le secret est accessible depuis la VM via `curl + JWT`
-  Les fichiers sont organisés (`keyvault.tf`, `variables.tf`, `outputs.tf`)

