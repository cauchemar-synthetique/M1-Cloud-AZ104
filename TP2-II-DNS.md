---

### 🌞 1. Donner un nom DNS à votre VM

**Objectif** : Ajouter un nom DNS à la VM via Terraform.

####  Modification du fichier `main.tf`

Dans la ressource `azurerm_public_ip`, ajout de la propriété suivante :

```hcl
resource "azurerm_public_ip" "main" {
  name                = "public-ip-cauchemar-tf"
  location            = azurerm_resource_group.main.location
  resource_group_name = azurerm_resource_group.main.name
  allocation_method   = "Static"
  sku                 = "Standard"
  domain_name_label   = "vmcauchemartf"  # ← nom DNS personnalisé
}
```

---

###  😒 2. Ajouter un output custom à Terraform

**Objectif** : Afficher l'IP publique et le FQDN dans la sortie de `terraform apply`.

####  Fichier `outputs.tf` créé :

```hcl
output "public_ip_address" {
  value       = azurerm_public_ip.main.ip_address
  description = "Adresse IP publique de la VM"
}

output "fqdn" {
  value       = azurerm_public_ip.main.fqdn
  description = "Nom de domaine DNS de la VM"
}
```

---

### 🌞 3. Proofs ! 📸

####  Sortie de la commande `terraform apply` :

```bash
Apply complete! Resources: 0 added, 0 changed, 0 destroyed.

Outputs:

public_ip_address = "172.167.170.33"
fqdn = "vmcauchemartf.uksouth.cloudapp.azure.com"
```

####  Connexion SSH réussie avec le nom DNS (agent SSH) :

```bash
ssh cauchemar34@vmcauchemartf.uksouth.cloudapp.azure.com
```

 Résultat : connexion réussie sans mot de passe.

---

###  Fichiers Terraform utilisés

- `main.tf` 
- `network.tf` 
- `outputs.tf` 
- (Autres fichiers éventuels : `variables.tf`, `provider.tf`, etc.)

