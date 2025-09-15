## IV. Monitoring

### 🌞 1. Intro

Azure propose des solutions de monitoring intégrées, notamment les *Platform Metrics* et *Logs Analytics Workspace*.  
Pour ce TP, nous utilisons les Platform Metrics car plus simples à intégrer avec Terraform.

### 🌞 2. Une alerte CPU

Nous avons créé un fichier `monitoring.tf` contenant les ressources suivantes :

- `azurerm_monitor_action_group.main`
- `azurerm_monitor_metric_alert.cpu_alert`

Seuil déclencheur : **CPU > 70%**

```hcl
resource "azurerm_monitor_action_group" "main" {
  name                = "ag-akatsuki-tf-alerts"
  resource_group_name = azurerm_resource_group.main.name
  short_name          = "vm-alerts"

  email_receiver {
    name          = "admin"
    email_address = var.alert_email_address
  }
}

resource "azurerm_monitor_metric_alert" "cpu_alert" {
  name                = "cpu-alert-${azurerm_linux_virtual_machine.main.name}"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_linux_virtual_machine.main.id]
  description         = "Alert when CPU usage exceeds 70%"
  severity            = 2

  criteria {
    metric_namespace = "Microsoft.Compute/virtualMachines"
    metric_name      = "Percentage CPU"
    aggregation      = "Average"
    operator         = "GreaterThan"
    threshold        = 70
  }

  window_size   = "PT5M"
  frequency     = "PT1M"
  auto_mitigate = true

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}
```

---

### 🌞 3. Alerte mémoire

Une deuxième alerte a été ajoutée dans `monitoring.tf` :

- `azurerm_monitor_metric_alert.ram_alert`

Seuil déclencheur : **RAM disponible < 512 Mo**

```hcl
resource "azurerm_monitor_metric_alert" "ram_alert" {
  name                = "ram-alert-${azurerm_linux_virtual_machine.main.name}"
  resource_group_name = azurerm_resource_group.main.name
  scopes              = [azurerm_linux_virtual_machine.main.id]
  description         = "Alert when available memory is below 512MB"
  severity            = 2

  criteria {
    metric_namespace = "Microsoft.Compute/virtualMachines"
    metric_name      = "Available Memory Bytes"
    aggregation      = "Average"
    operator         = "LessThan"
    threshold        = 536870912
  }

  window_size   = "PT5M"
  frequency     = "PT1M"
  auto_mitigate = true

  action {
    action_group_id = azurerm_monitor_action_group.main.id
  }
}
```

---

### 🌞 4. Proofs

#### A. Liste des alertes via CLI

```bash
az monitor metrics alert list -g akatsuki-tf --query "[].{Name:name, Description:description}" -o table
```

**Sortie :**

```
Name                          Description
----------------------------  ------------------------------------------
cpu-alert-vm-cauchemar-tf     Alert when CPU usage exceeds 70%
ram-alert-vm-cauchemar-tf     Alert when available memory is below 512MB
```

#### B. Stress de la VM pour déclencher les alertes

```bash
# Installer l'outil
sudo apt install stress-ng -y

# Stress CPU
stress-ng --cpu 1 --timeout 600s

# Stress RAM
stress-ng --vm 1 --vm-bytes 700M --timeout 600s
```

>  Après quelques minutes, j'ai reçu **une alerte email** sur mon smartphone.  
>  Les alertes sont également visibles dans le portail Azure > VM > Monitoring > Alerts

