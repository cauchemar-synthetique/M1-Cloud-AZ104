# Résumé des commandes – TP2 Azure NSG avec Terraform

## 📅 Date
TP réalisé le : **2025-09-15**

---

## 🔧 Connexion à la VM

```bash
ssh cauchemar34@172.167.170.33
```
✅ Connexion réussie (port 22, avant changement du port SSH).

---

## 🔍 Vérification du port d'écoute SSH après modification (port 2222)

```bash
sudo ss -tnlp | grep sshd
```
```
LISTEN 0      128          0.0.0.0:2222      0.0.0.0:*    users:(("sshd",pid=1473,fd=3))
LISTEN 0      128             [::]:2222         [::]:*    users:(("sshd",pid=1473,fd=4))
```

✅ Le serveur OpenSSH écoute bien sur le port **2222**.

---

## 🛡️ Vérification du NSG et de la VM

```bash
az vm show --name vm-cauchemar-tf --resource-group akatsuki-tf --show-details --output json
```
🔎 Résultat : VM `vm-cauchemar-tf` avec NSG associé et IP publique `172.167.170.33`.

Extrait :
```json
"networkProfile": {
  "networkInterfaces": [
    {
      "id": "/subscriptions/.../networkInterfaces/nic-cauchemar-tf",
      "primary": true,
      "resourceGroup": "akatsuki-tf"
    }
  ]
},
"publicIps": "172.167.170.33",
...
```

---

## ❌ Tentative de connexion sur port bloqué (2222)

```bash
ssh -p 2222 cauchemar34@172.167.170.33
```
```
Connection closed by 172.167.170.33 port 2222
```

✅ Le port 2222 est **bloqué** par le NSG — preuve que le filtrage fonctionne.

---

## ✅ `terraform apply` (modification du NSG)

```bash
terraform apply
```

Extrait :
```
azurerm_network_security_group.main: Modifying... 
azurerm_network_security_group.main: Modifications complete after 3s
Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

✅ NSG modifié avec succès pour n'autoriser que le port **22** depuis ton IP publique.
