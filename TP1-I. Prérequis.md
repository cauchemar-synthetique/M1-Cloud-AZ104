
# TP Cloud – Partie 1 & 2 : Prérequis

## 1. Starting blocks

➜ **Activez votre compte Azure for Students**

 Activation réalisée via le site officiel : [https://azure.microsoft.com/fr-fr/free/students/](https://azure.microsoft.com/fr-fr/free/students/)  
 Connexion avec compte EFREI réussie

➜ **Installer le Azure CLI az sur votre poste**

 Installation via `winget` :
```powershell
winget install --id Microsoft.AzureCLI -e
```

 Connexion :
```powershell
az login
```

 Vérification de l’abonnement :
```powershell
az account show --output table
```

 Résultat :
```
Azure for Students  | Enabled | efrei.net | Efrei
```

➜ **Installer Terraform sur votre poste**

 Installation manuelle via PowerShell :

```powershell
New-Item -Path "C:\Tools\Terraform" -ItemType Directory -Force
Invoke-WebRequest -Uri "https://releases.hashicorp.com/terraform/1.8.5/terraform_1.8.5_windows_amd64.zip" `
  -OutFile "C:\Tools\Terraform\terraform.zip"
Expand-Archive -Path "C:\Tools\Terraform\terraform.zip" -DestinationPath "C:\Tools\Terraform" -Force
Remove-Item "C:\Tools\Terraform\terraform.zip"
```

 Ajout au PATH :
```powershell
$terraformPath = "C:\Tools\Terraform"
$currentPath = [Environment]::GetEnvironmentVariable("Path", "User")
if ($currentPath -notlike "*$terraformPath*") {
    [Environment]::SetEnvironmentVariable("Path", "$currentPath;$terraformPath", "User")
}
$env:Path = [System.Environment]::GetEnvironmentVariable("Path","User") + ";" +
            [System.Environment]::GetEnvironmentVariable("Path","Machine")
terraform -version
```

 Résultat :
```
Terraform v1.8.5
on windows_amd64
```

>  *Note pour les Windowsiens : installation réalisée en PowerShell natif.*

---

## 2. Une paire de clés SSH

### 🌞 A. Choix de l’algorithme de chiffrement

**Vous n’utiliserez PAS RSA.**

 *Pourquoi éviter RSA ?*  
 RSA devient moins recommandé pour les connexions SSH à cause de sa lenteur et de la taille croissante des clés requises pour garantir un bon niveau de sécurité.  
 Source : [ANSSI - Recommandations cryptographiques 2022](https://www.ssi.gouv.fr/administration/bonnes-pratiques/recommandations-cryptographiques/)

 *Quel algorithme utiliser ?*  
 Recommandation : **ED25519** – moderne, rapide, sécurisé, et supporté par OpenSSH.  
 Source : [OpenSSH - ssh-keygen man page](https://man.openbsd.org/ssh-keygen#t)

### 🌞 B. Génération de votre paire de clés

 Commande utilisée :
```powershell
ssh-keygen -t ed25519 -f "$env:USERPROFILE\.ssh\cloud_tp1" -C "edgard@efrei.net"
```

 Fichiers générés :
```
C:\Users\33749\.ssh\cloud_tp1       (clé privée protégée par mot de passe)
C:\Users\33749\.ssh\cloud_tp1.pub   (clé publique)
```

 Vérification :
```powershell
Get-ChildItem "$env:USERPROFILE\.ssh\cloud_tp1*"
```

### 🌞 C. Agent SSH – configuration

 Activation de l’agent SSH (en administrateur) :
```powershell
Start-Service ssh-agent
Set-Service -Name ssh-agent -StartupType Automatic
```

 Ajout de la clé dans l’agent (en session utilisateur) :
```powershell
ssh-add "$env:USERPROFILE\.ssh\cloud_tp1"
```

 Sortie :
```
Enter passphrase...
Identity added: C:\Users\33749\.ssh\cloud_tp1 (edgard@efrei.net)
```

 Vérification de la clé chargée :
```powershell
ssh-add -l
```

Sortie attendue :
```
256 SHA256:xxxxxxxxxxxxxxxxxxxxxxxx edgard@efrei.net (ED25519)
```

*Note : une fois la clé ajoutée, elle est disponible pour toutes les connexions SSH durant la session.*

---
