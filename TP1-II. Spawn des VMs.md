# 📘 TP Azure – VM avec CLI et Terraform 
**TP** : Création d’une VM Azure via CLI (`az`) puis Terraform

---

##  Partie 1 – Création de VM via Azure CLI

###  Création du groupe de ressources

```bash
az group create --location uksouth --name akatsuki
```

###  Création de la VM `vm-cauchemar`

```bash
az vm create   --resource-group akatsuki   --name vm-cauchemar   --image Ubuntu2204   --admin-username cauchemar34   --ssh-key-values ~/.ssh/id_ed25519.pub
```

###  Connexion SSH (agent SSH activé, pas de mot de passe demandé)

```bash
ssh cauchemar34@<IP_PUBLIQUE_DONNÉE>
```

---

##  Partie 2 – Création de VM avec Terraform

###  Arborescence

```bash
C:\Users\33749\Documents\TP_AZURE_TERRAFORM
├── main.tf
├── cloud_tp1.pub
└── .terraform.lock.hcl
```

###  Contenu du `main.tf`

Voir fichier rendu séparément

---

##  Commandes Terraform exécutées

### Initialisation

```bash
terraform init
# Output :
# Terraform has been successfully initialized!
```

### Validation

```bash
terraform validate
# Output :
# Success! The configuration is valid.
```

### Planification

```bash
terraform plan
# Output :
# Plan: 3 to add, 0 to change, 0 to destroy.
```

### Déploiement

```bash
terraform apply
# Output (résumé) :
# Apply complete! Resources: 3 added, 0 changed, 0 destroyed.
```

### Affichage de l’adresse IP publique de la VM

```bash
az vm list-ip-addresses --name vm-cauchemar-tf --resource-group akatsuki-tf --output table
# Output :
# VirtualMachine    PublicIPAddresses    PrivateIPAddresses
# ----------------  -------------------  --------------------
# vm-cauchemar-tf   172.167.170.33       10.0.1.4
```

### Connexion SSH à la VM créée via Terraform

```bash
ssh cauchemar34@172.167.170.33
# Output :
# Welcome to Ubuntu 22.04.5 LTS ...
# ...
# cauchemar34@vmcauchemartf:~$
```

---

## 🛠️ Dépannage effectué

- 🔐 Ajout d’un **Network Security Group** avec autorisation du port **22**
- ⚠️ Correction d’erreurs Terraform :
  - `"network_security_group_id" is not expected here` corrigé en créant une association explicite
  - `"PublicIPCountLimitReached"` résolu en supprimant les IPs inutilisées
  - Correction de `sku` et `allocation_method` dans `azurerm_public_ip`

---

##  Vérifications des services sur la VM

```bash
systemctl status walinuxagent.service
systemctl status cloud-init.service
cloud-init status
```

Tous les services sont présents et actifs 

---

## 🧹 Commande optionnelle de nettoyage

```bash
terraform destroy
```

