
# TP Cloud – Partie 1 & 2

👤 **Utilisateur** : Edgard B. (EFREI)  
🖥️ **Système** : Windows PowerShell  

---

## ✅ PARTIE 1 – Starting Blocks

### 🔹 1. Connexion à Azure CLI

```powershell
az login
```

**Sortie (extrait)** :
```
[1] *  Azure for Students   532ae6c0-cf53-4b9b-927a-bfc72f275d15  Efrei
```

```powershell
az account show --output table
```

**Sortie :**
```
Azure for Students  | Enabled | efrei.net | Efrei
```

### 🔹 2. Préparation de Terraform

```powershell
New-Item -Path "C:\Tools\Terraform" -ItemType Directory -Force
```

```powershell
Invoke-WebRequest -Uri "https://releases.hashicorp.com/terraform/1.8.5/terraform_1.8.5_windows_amd64.zip" `
  -OutFile "C:\Tools\Terraform\terraform.zip"
```

```powershell
Expand-Archive -Path "C:\Tools\Terraform\terraform.zip" -DestinationPath "C:\Tools\Terraform" -Force
Remove-Item "C:\Tools\Terraform\terraform.zip"
```

#### Ajout au PATH utilisateur

```powershell
$terraformPath = "C:\Tools\Terraform"
$currentPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($currentPath -notlike "*$terraformPath*") {
    [Environment]::SetEnvironmentVariable("Path", "$currentPath;$terraformPath", "User")
}
```

#### Rafraîchir le PATH dans la session

```powershell
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","User") + ";" +
            [System.Environment]::GetEnvironmentVariable("Path","Machine")
```

```powershell
terraform -version
```

**Sortie :**
```
Terraform v1.8.5
on windows_amd64
```

---

## ✅ PARTIE 2 – Clé SSH & Agent SSH

### 🔐 Génération de la paire de clés SSH `ed25519`

```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\cloud_tp1" -C "edgard@efrei.net"
```

**Fichiers générés** :
- `C:\Users\33749\.ssh\cloud_tp1`
- `C:\Users\33749\.ssh\cloud_tp1.pub`

### 📁 Vérification de la paire de clés

```powershell
Get-ChildItem "$env:USERPROFILE\.ssh\cloud_tp1*"
```

### 🤖 Activer l’agent SSH (PowerShell admin)

```powershell
Start-Service ssh-agent
Set-Service -Name ssh-agent -StartupType Automatic
```

### ➕ Ajouter la clé à l’agent SSH

```powershell
ssh-add "$env:USERPROFILE\.ssh\cloud_tp1"
```

**Sortie :**
```
Enter passphrase...
Identity added: C:\Users\33749\.ssh\cloud_tp1 (edgard@efrei.net)
```

### 🔍 Vérification

```powershell
ssh-add -l
```

**Sortie :**
```
256 SHA256:xxxxxxxxxxxxxxxxxxxxxxxx edgard@efrei.net (ED25519)
```

---

✅ **Configuration terminée pour les parties 1 et 2 !**
