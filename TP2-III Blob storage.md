## 1.  Intro : Blob Storage dans Azure

Azure permet de créer un **Storage Account** contenant des **containers Blob** pour stocker des fichiers. Ces containers peuvent être accédés depuis une VM Azure autorisée grâce à un **rôle IAM** comme `Storage Blob Data Contributor`. Idéal pour des **backups** ou du **partage de fichiers entre VMs**.

---

## 2.  Let's Go : Déploiement via Terraform

###  Fichiers utilisés

- `main.tf`
- `storage.tf`
- `variables.tf` *(optionnel si utilisation de `var.storage_account_name` etc.)*

###  Fichier `storage.tf`

```hcl
resource "azurerm_storage_account" "main" {
  name                     = var.storage_account_name
  resource_group_name      = azurerm_resource_group.main.name
  location                 = azurerm_resource_group.main.location
  account_tier             = "Standard"
  account_replication_type = "LRS"
}

resource "azurerm_storage_container" "meowcontainer" {
  name                  = var.storage_container_name
  storage_account_id    = azurerm_storage_account.main.id
  container_access_type = "private"
}

data "azurerm_virtual_machine" "main" {
  name                = azurerm_linux_virtual_machine.main.name
  resource_group_name = azurerm_resource_group.main.name
}

resource "azurerm_role_assignment" "vm_blob_access" {
  principal_id         = data.azurerm_virtual_machine.main.identity[0].principal_id
  role_definition_name = "Storage Blob Data Contributor"
  scope                = azurerm_storage_account.main.id

  depends_on = [
    azurerm_linux_virtual_machine.main
  ]
}
```

###  Ajout dans `main.tf`

Bloc ajouté dans `azurerm_linux_virtual_machine.main` :

```hcl
identity {
  type = "SystemAssigned"
}
```

###  Commandes Terraform exécutées

```bash
terraform init
terraform validate
terraform apply
```

 **Résultat :**

```
Apply complete! Resources: 4 added, 2 changed, 0 destroyed.

Outputs:
fqdn = "vmcauchemartf.uksouth.cloudapp.azure.com"
public_ip_address = "172.167.170.33"
```

---

## 3. Proooooofs depuis la VM

###  Installation de AzCopy

```bash
wget https://aka.ms/downloadazcopy-v10-linux -O azcopy.tar.gz
tar -xf azcopy.tar.gz
sudo cp ./azcopy_linux_amd64_*/azcopy /usr/bin/
azcopy --version
```

###  Connexion via Managed Identity

```bash
azcopy login --identity
```

 **Réponse :**
```
INFO: Logging in under the identity of the VM
```

###  Upload d’un fichier

```bash
echo "cauchemar_backup" > backup.txt

azcopy copy backup.txt "https://stcauchemarXXXX.blob.core.windows.net/meowcontainer/backup.txt"
```

###  Téléchargement du fichier

```bash
azcopy copy "https://stcauchemarXXXX.blob.core.windows.net/meowcontainer/backup.txt" ./downloaded_backup.txt
cat downloaded_backup.txt
```

 **Affichage :**
```
cauchemar_backup
```

---

## 4.  Fonctionnement de `azcopy login --identity`

`azcopy` utilise le **Managed Identity** pour contacter le service de métadonnées d’Azure (IMDS) et récupérer un **JWT (Jeton OAuth2)**. Ce jeton est ensuite utilisé pour authentifier les requêtes auprès d’Azure Blob Storage.

---

## 5.  Obtenir manuellement un JWT

###  Commande depuis la VM

```bash
curl -H "Metadata:true" "http://169.254.169.254/metadata/identity/oauth2/token?api-version=2018-02-01&resource=https://storage.azure.com/"
```

 **Extrait de réponse :**
```json
{
  "access_token": "eyJ0eXAiOiJKV1QiLCJhbGciOi...",
  "expires_on": "1694899109",
  "resource": "https://storage.azure.com/",
  
}
```

---

## 6.  Pourquoi l’IP `169.254.169.254` est-elle joignable ?

Cette adresse IP est une **adresse spéciale de lien local (link-local)** accessible **sans passerelle**, toujours disponible pour toutes les VMs Azure. Azure ajoute une **route locale automatique** dans la table de routage réseau de la VM permettant la communication directe avec le service IMDS (Azure Instance Metadata Service).

---

 **Fichiers finaux :**

```
 TP_AZURE_TERRAFORM/
├── main.tf
├── network.tf
├── outputs.tf
├── storage.tf
└── variables.tf (si utilisé)
```


