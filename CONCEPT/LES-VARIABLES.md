Dans TF les variables constituent le coeur du système de paramétrage.
C'est à dire qu'elles permettent de rendre les configurations flexibles et réutilisables
Il faut savoir qu'elles se déclinent en 3 catégories principales selon leur moment d'utilisation dans le cycle de vie.

## Les types :

### Input
**Définition** : Paramètres fournis à Terraform avant l'exécution pour personnaliser le comportement de la configuration.

**Déclaration dans variables.tfvars** :

```hcl
variable "name_label" {
  type        = string
  description = "Description de la variable"
  default     = "valeur_par_defaut"
  sensitive   = false
  validation {
    condition     = length(var.name_label) > 0
    error_message = "Le nom ne peut pas être vide."
  }
}
```

**Attributs disponibles** :
- `type` : Type de données (obligatoire)
- `description` : Documentation de la variable
- `default` : Valeur par défaut si non fournie
- `sensitive` : Masque la valeur dans les logs
- `validation` : Règles de validation personnalisées

**Utilisation** :
```hcl
resource "aws_instance" "web" {
  instance_type = var.name_label
}
```

### Variables locales
**Définition** : Elles sont calculées dynamiquement dans la configuration au moment l'exécution.

**Déclaration** :
```hcl
locals {
  instance_name = "${var.project_name}-${var.environment}-web"
  common_tags = {
    Project     = var.project_name
    Environment = var.environment
    ManagedBy   = "terraform"
  }
}
```

**Utilisation** :
```hcl
resource "aws_instance" "web" {
  tags = local.common_tags
  
  user_data = <<-EOF
    #!/bin/bash
    echo "Instance: ${local.instance_name}" > /tmp/info.txt
  EOF
}
```

**Avantages** :
- Évitent la répétition de code
- Permettent des calculs complexes
- Centralisent la logique métier

### Variables Output
**Définition** : Valeurs extraites et afficher dans le terminal après l'exécution de la config.

**Déclaration dans `outputs.tf`** :
```hcl
output "aws_instance_public_dns" {
  value       = "http://${aws_instance.nginx1.public_dns}"
  description = "Public DNS hostname for the EC2 instance"
  sensitive   = false
}

output "database_connection_string" {
  value       = "postgresql://${aws_db_instance.main.username}:${var.db_password}@${aws_db_instance.main.endpoint}"
  description = "Database connection string"
  sensitive   = true
}
```

**Consultation** :
```bash
terraform output
terraform output aws_instance_public_dns
terraform output -json
```




## Formes de variables

Comme beaucoup de language on peut déclarer plusieurs formes de variables pour différents usages qu'on va voir à travers différents exemples, mais avant listons les pour bien comprendre leur différences : 

### Primitifs
```hcl
variable "server_name" {
  type = string
  default = "web-server"
}

variable "server_count" {
  type = number
  default = 2
}

variable "enable_monitoring" {
  type = bool
  default = true
}
```

### Collections
```hcl
# Liste
variable "availability_zones" {
  type = list(string)
  default = ["us-east-1a", "us-east-1b"]
}

# Set (valeurs uniques)
variable "security_groups" {
  type = set(string)
  default = ["sg-web", "sg-ssh"]
}

# Map
variable "instance_types" {
  type = map(string)
  default = {
    dev  = "t2.micro"
    prod = "t3.large"
  }
}
```

### Structurels
```hcl
# Object
variable "server_config" {
  type = object({
    name          = string
    instance_type = string
    disk_size     = number
    monitoring    = bool
  })
  default = {
    name          = "web-server"
    instance_type = "t2.micro"
    disk_size     = 20
    monitoring    = true
  }
}

# Tuple
variable "server_ports" {
  type = tuple([string, number, bool])
  default = ["HTTP", 80, true]
}
```

### Spéciaux
```hcl
# Any - accepte n'importe quel type
variable "flexible_config" {
  type = any
}

# Null - absence de valeur
variable "optional_parameter" {
  type = string
  default = null
}
```

## Méthodes d'assignation

### 1. Fichiers de variables (.tfvars)
```hcl
# terraform.tfvars
project_name = "my-app"
environment = "production"
instance_count = 3
```

### 2. Ligne de commande
```bash
terraform apply -var="project_name=my-app" -var="environment=prod"
terraform apply -var-file="production.tfvars"
```

### 3. Variables d'environnement
```bash
export TF_VAR_project_name="my-app"
export TF_VAR_environment="production"
terraform apply
```

### 4. Interface interactive
```bash
terraform apply
# Terraform demandera les valeurs manquantes
```

## Sécurisation des données sensibles

### ❌ Mauvaise pratique
```hcl
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
  region     = "us-east-1"
}
```

### ✅ Bonne pratique

**1. Variables d'environnement** :
```bash
export TF_VAR_aws_access_key="AKIAIOSFODNN7EXAMPLE"
export TF_VAR_aws_secret_key="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
```

**2. Configuration Terraform** :
```hcl
variable "aws_access_key" {
  type      = string
  sensitive = true
}

variable "aws_secret_key" {
  type      = string
  sensitive = true
}

provider "aws" {
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  region     = "us-east-1"
}
```

**3. Alternatives recommandées** :
```hcl
# Utilisation des profils AWS CLI
provider "aws" {
  profile = "production"
  region  = "us-east-1"
}

# Utilisation des rôles IAM
provider "aws" {
  assume_role {
    role_arn = "arn:aws:iam::123456789012:role/TerraformRole"
  }
}
```

## Validation et contraintes

### Validation simple
```hcl
variable "environment" {
  type = string
  validation {
    condition = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}
```

### Validation complexe
```hcl
variable "instance_config" {
  type = object({
    type = string
    size = number
  })
  validation {
    condition = var.instance_config.size >= 10 && var.instance_config.size <= 1000
    error_message = "Instance size must be between 10 and 1000 GB."
  }
  validation {
    condition = contains(["t2.micro", "t3.small", "t3.medium"], var.instance_config.type)
    error_message = "Instance type must be t2.micro, t3.small, or t3.medium."
  }
}
```

## Bonnes pratiques

### Organisation des fichiers
```
├── variables.tf      # Déclarations des variables input
├── locals.tf         # Définitions des variables locales
├── outputs.tf        # Déclarations des outputs
├── terraform.tfvars  # Valeurs par défaut
├── dev.tfvars       # Valeurs pour l'environnement dev
└── prod.tfvars      # Valeurs pour l'environnement prod
```

### Conventions de nommage
```hcl
# Variables input : snake_case
variable "vpc_cidr_block" { }
variable "enable_dns_hostnames" { }

# Variables locales : snake_case avec préfixe logique
locals {
  network_name = "${var.project_name}-${var.environment}-vpc"
  common_tags = {
    Project = var.project_name
    Environment = var.environment
  }
}

# Outputs : descriptifs et explicites
output "vpc_id" { }
output "public_subnet_ids" { }
output "load_balancer_dns_name" { }
```

## Exemples Pratiques

### Configuration Multi-environnement
```hcl
# variables.tf
variable "environment" {
  type = string
  validation {
    condition = contains(["dev", "staging", "prod"], var.environment)
    error_message = "Environment must be dev, staging, or prod."
  }
}

# locals.tf
locals {
  instance_configs = {
    dev = {
      instance_type = "t2.micro"
      min_size     = 1
      max_size     = 2
    }
    staging = {
      instance_type = "t3.small"
      min_size     = 2
      max_size     = 4
    }
    prod = {
      instance_type = "t3.medium"
      min_size     = 3
      max_size     = 10
    }
  }
  
  current_config = local.instance_configs[var.environment]
}

# main.tf
resource "aws_autoscaling_group" "web" {
  min_size         = local.current_config.min_size
  max_size         = local.current_config.max_size
  desired_capacity = local.current_config.min_size
  
  launch_template {
    id      = aws_launch_template.web.id
    version = "$Latest"
  }
}
```