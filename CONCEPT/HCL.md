# HCL - HashiCorp Configuration Language - Fiche Pédagogique

## Vue d'ensemble

HCL (HashiCorp Configuration Language) est un langage de configuration déclaratif développé par HashiCorp. Il s'agit d'une version simplifiée et plus lisible du JSON, spécialement conçue pour la configuration d'infrastructure et la lisibilité humaine.

## Caractéristiques Fondamentales

### Philosophie du Langage
- **Lisibilité** : Syntaxe proche du langage naturel
- **Simplicité** : Structure claire et intuitive
- **Expressivité** : Support des expressions complexes
- **Compatibilité** : Peut lire et traiter JSON natif
- **Interactivité** : Support des arguments en ligne de commande

### Point d'Entrée
Toute configuration Terraform commence par un fichier `main.tf` qui sert de point d'entrée principal pour la configuration.

## Structure Syntaxique

### Anatomie d'un Block
```hcl
<BLOCK_TYPE> "<LABEL_1>" "<LABEL_2>" {
  <ARGUMENT_NAME> = <ARGUMENT_VALUE>
  <ARGUMENT_NAME> = <ARGUMENT_VALUE>
  
  <NESTED_BLOCK_NAME> {
    <NESTED_ARGUMENT> = <VALUE>
  }
}
```

**Composants** :
1. **BLOCK_TYPE** : Mot-clé définissant le type d'objet
2. **LABELS** : Étiquettes d'identification (type, nom)
3. **ARGUMENTS** : Paires clé-valeur de configuration
4. **NESTED_BLOCKS** : Blocs imbriqués pour configuration avancée

### Exemple Concret
```hcl
resource "aws_instance" "web_server" {
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  
  tags = {
    Name        = "WebServer"
    Environment = "Production"
  }
  
  vpc_security_group_ids = ["sg-0123456789abcdef0"]
  
  user_data = <<-EOF
    #!/bin/bash
    yum update -y
    yum install -y httpd
    systemctl start httpd
  EOF
  
  ebs_block_device {
    device_name = "/dev/sda1"
    volume_size = 20
    volume_type = "gp3"
  }
}
```

## Types de Blocks Terraform

### 1. Block Resource
```hcl
resource "<PROVIDER>_<TYPE>" "<NAME>" {
  # Configuration de la ressource
}
```

**Exemple** :
```hcl
resource "aws_s3_bucket" "data_lake" {
  bucket = "my-company-data-lake"
  
  versioning {
    enabled = true
  }
  
  server_side_encryption_configuration {
    rule {
      apply_server_side_encryption_by_default {
        sse_algorithm = "AES256"
      }
    }
  }
}
```

### 2. Block Data Source
```hcl
data "<PROVIDER>_<TYPE>" "<NAME>" {
  # Critères de recherche
}
```

**Exemple** :
```hcl
data "aws_ami" "ubuntu" {
  most_recent = true
  owners      = ["099720109477"] # Canonical
  
  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }
  
  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}
```

### 3. Block Provider
```hcl
provider "<PROVIDER_NAME>" {
  # Configuration du provider
}
```

**Exemple** :
```hcl
provider "aws" {
  region  = "us-west-2"
  profile = "production"
  
  default_tags {
    tags = {
      Project   = "MyApp"
      Owner     = "DevOps Team"
      ManagedBy = "Terraform"
    }
  }
}
```

### 4. Block Variable
```hcl
variable "<NAME>" {
  # Configuration de la variable
}
```

**Exemple** :
```hcl
variable "instance_count" {
  type        = number
  description = "Number of EC2 instances to create"
  default     = 2
  
  validation {
    condition     = var.instance_count >= 1 && var.instance_count <= 10
    error_message = "Instance count must be between 1 and 10."
  }
}
```

### 5. Block Output
```hcl
output "<NAME>" {
  # Configuration de la sortie
}
```

**Exemple** :
```hcl
output "instance_public_ips" {
  description = "Public IP addresses of the instances"
  value       = aws_instance.web[*].public_ip
  sensitive   = false
}
```

### 6. Block Locals
```hcl
locals {
  # Variables calculées localement
}
```

**Exemple** :
```hcl
locals {
  environment = terraform.workspace
  
  common_tags = {
    Environment = local.environment
    Project     = var.project_name
    ManagedBy   = "terraform"
    CreatedAt   = timestamp()
  }
  
  instance_name = "${var.project_name}-${local.environment}-web"
}
```

## Système de Référencement

### Référencement de Ressources
```hcl
<TYPE_RESSOURCE>.<LABEL>.<ATTRIBUT>
```

**Exemples** :
```hcl
# Référencer l'ID d'une instance
resource "aws_security_group" "web" {
  name_prefix = "web-"
  
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }
}

resource "aws_instance" "web_server" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  vpc_security_group_ids = [aws_security_group.web.id]
  
  tags = merge(
    local.common_tags,
    {
      Name = local.instance_name
    }
  )
}

# Sortie utilisant les références
output "web_server_details" {
  value = {
    instance_id = aws_instance.web_server.id
    public_ip   = aws_instance.web_server.public_ip
    public_dns  = aws_instance.web_server.public_dns
  }
}
```

### Référencement de Data Sources
```hcl
data.<TYPE_DATA>.<LABEL>.<ATTRIBUT>
```

**Exemple** :
```hcl
data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

resource "aws_instance" "app" {
  ami           = data.aws_ami.ubuntu.id
  instance_type = "t2.micro"
  subnet_id     = data.aws_subnets.default.ids[0]
}
```

## Types de Données et Valeurs

### Types Primitifs
```hcl
# String
server_name = "web-server-01"
description = <<-EOT
  Multi-line string
  with preserved formatting
EOT

# Number
port_number = 8080
cpu_count   = 4

# Boolean
enable_monitoring = true
public_access    = false
```

### Collections
```hcl
# List
availability_zones = ["us-west-2a", "us-west-2b", "us-west-2c"]
allowed_ports     = [22, 80, 443]

# Set
unique_tags = toset(["web", "api", "database"])

# Map
instance_types = {
  small  = "t2.micro"
  medium = "t3.small"
  large  = "t3.medium"
}
```

### Objets Complexes
```hcl
server_config = {
  name     = "web-server"
  type     = "t2.micro"
  storage  = 20
  backup   = true
  networks = ["public", "private"]
}
```

## Expressions et Fonctions

### Expressions Conditionnelles
```hcl
instance_type = var.environment == "production" ? "t3.large" : "t2.micro"

tags = var.environment == "production" ? {
  Environment = "prod"
  Backup     = "daily"
} : {
  Environment = "dev"
  Backup     = "none"
}
```

### Fonctions Intégrées
```hcl
# Fonctions de string
upper_name = upper(var.project_name)
formatted  = format("server-%02d", count.index + 1)

# Fonctions de collection
all_zones    = join(",", data.aws_availability_zones.available.names)
zone_count   = length(data.aws_availability_zones.available.names)
first_zone   = element(data.aws_availability_zones.available.names, 0)

# Fonctions de map
merged_tags = merge(local.common_tags, var.additional_tags)
tag_keys   = keys(local.common_tags)

# Fonctions de fichier
user_data = file("${path.module}/scripts/install.sh")
config    = templatefile("${path.module}/templates/config.tpl", {
  database_url = aws_db_instance.main.endpoint
  app_port    = var.app_port
})
```

### Opérateurs
```hcl
# Arithmétiques
total_storage = var.base_storage + var.additional_storage
cpu_ratio    = var.cpu_count / 2

# Logiques
is_production = var.environment == "prod" && var.high_availability == true
needs_backup  = var.environment != "dev" || var.force_backup == true

# Comparaison
is_large_instance = var.instance_size >= 4
is_valid_env     = contains(["dev", "stage", "prod"], var.environment)
```

## Blocks Imbriqués et Configurations Complexes

### Exemple avec Blocks Imbriqués
```hcl
resource "aws_launch_template" "web" {
  name_prefix   = "${var.project_name}-"
  image_id      = data.aws_ami.ubuntu.id
  instance_type = var.instance_type
  
  vpc_security_group_ids = [aws_security_group.web.id]
  
  user_data = base64encode(templatefile("${path.module}/user-data.sh", {
    app_name = var.app_name
    env      = var.environment
  }))
  
  block_device_mappings {
    device_name = "/dev/sda1"
    
    ebs {
      volume_size = var.root_volume_size
      volume_type = "gp3"
      iops        = 3000
      throughput  = 125
      encrypted   = true
      
      delete_on_termination = true
    }
  }
  
  network_interfaces {
    associate_public_ip_address = true
    security_groups            = [aws_security_group.web.id]
    delete_on_termination      = true
  }
  
  monitoring {
    enabled = var.detailed_monitoring
  }
  
  tag_specifications {
    resource_type = "instance"
    
    tags = merge(
      local.common_tags,
      {
        Name = "${var.project_name}-web"
        Type = "web-server"
      }
    )
  }
}
```

### Configuration avec Listes d'Objets
```hcl
variable "security_group_rules" {
  type = list(object({
    type        = string
    from_port   = number
    to_port     = number
    protocol    = string
    cidr_blocks = list(string)
    description = string
  }))
  
  default = [
    {
      type        = "ingress"
      from_port   = 80
      to_port     = 80
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTP access"
    },
    {
      type        = "ingress"
      from_port   = 443
      to_port     = 443
      protocol    = "tcp"
      cidr_blocks = ["0.0.0.0/0"]
      description = "HTTPS access"
    }
  ]
}

resource "aws_security_group" "web" {
  name_prefix = "${var.project_name}-web-"
  
  dynamic "ingress" {
    for_each = [
      for rule in var.security_group_rules : rule
      if rule.type == "ingress"
    ]
    
    content {
      from_port   = ingress.value.from_port
      to_port     = ingress.value.to_port
      protocol    = ingress.value.protocol
      cidr_blocks = ingress.value.cidr_blocks
      description = ingress.value.description
    }
  }
}
```

## Commentaires et Documentation

### Types de Commentaires
```hcl
# Commentaire sur une ligne

/*
  Commentaire
  sur plusieurs
  lignes
*/

// Style de commentaire alternatif

resource "aws_instance" "web" {
  # Configuration de l'instance web principale
  ami           = data.aws_ami.ubuntu.id  # AMI Ubuntu LTS
  instance_type = var.instance_type       # Type défini par variable
  
  /*
   * Configuration des tags
   * Applique les tags communs + tags spécifiques
   */
  tags = merge(
    local.common_tags,
    {
      Name = "WebServer"  // Nom de l'instance
      Role = "Frontend"   // Rôle dans l'architecture
    }
  )
}
```

## Aspects Sécurité et Sensibilité

### Sensibilité à la Casse (Case Sensitive)
HCL est **sensible à la casse**, ce qui signifie que `myVariable`, `MyVariable` et `MYVARIABLE` sont des identifiants différents.

```hcl
# Ces trois variables sont distinctes
variable "database_name" {
  type = string
}

variable "Database_Name" {
  type = string  # Variable différente de la précédente
}

variable "DATABASE_NAME" {
  type = string  # Encore une variable différente
}

# Référencements - doivent respecter la casse exacte
resource "aws_db_instance" "main" {
  identifier = var.database_name     # ✅ Correct
  # identifier = var.Database_Name  # ❌ Erreur si la variable est "database_name"
}
```

**Implications pratiques** :
- Les noms de ressources : `aws_instance.Web` ≠ `aws_instance.web`
- Les attributs : `.public_ip` ≠ `.Public_IP`
- Les variables : `var.project_name` ≠ `var.Project_Name`

### Variables d'Environnement TF_VAR_

**Convention TF_VAR_** : Terraform reconnaît automatiquement les variables d'environnement préfixées par `TF_VAR_`.

```bash
# Variables d'environnement système
export TF_VAR_aws_access_key="AKIAIOSFODNN7EXAMPLE"
export TF_VAR_aws_secret_key="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLE"
export TF_VAR_environment="production"
export TF_VAR_instance_count="3"
```

```hcl
# Dans la configuration Terraform
variable "aws_access_key" {
  type      = string
  sensitive = true
}

variable "aws_secret_key" {
  type      = string
  sensitive = true
}

variable "environment" {
  type = string
}

variable "instance_count" {
  type = number
}

# Utilisation automatique des TF_VAR_
provider "aws" {
  access_key = var.aws_access_key  # Automatiquement = TF_VAR_aws_access_key
  secret_key = var.aws_secret_key  # Automatiquement = TF_VAR_aws_secret_key
}

resource "aws_instance" "web" {
  count         = var.instance_count  # Automatiquement = TF_VAR_instance_count
  instance_type = var.environment == "production" ? "t3.large" : "t2.micro"
}
```

**Mapping automatique** :
- `TF_VAR_database_password` → `var.database_password`
- `TF_VAR_vpc_cidr_block` → `var.vpc_cidr_block`
- **Attention** : La casse doit correspondre exactement

### Sécurisation avec TF_VAR_

**❌ Mauvaise pratique - Données sensibles dans le code** :
```hcl
provider "aws" {
  access_key = "AKIAIOSFODNN7EXAMPLE"
  secret_key = "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLE"
  region     = "us-east-1"
}

resource "aws_db_instance" "main" {
  password = "SuperSecretPassword123!"
}
```

**✅ Bonne pratique - Variables d'environnement** :
```bash
# Variables d'environnement
export TF_VAR_aws_access_key="AKIAIOSFODNN7EXAMPLE"
export TF_VAR_aws_secret_key="wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLE" 
export TF_VAR_db_password="SuperSecretPassword123!"
```

```hcl
# Configuration Terraform sécurisée
variable "aws_access_key" {
  type        = string
  description = "AWS Access Key ID"
  sensitive   = true
}

variable "aws_secret_key" {
  type        = string
  description = "AWS Secret Access Key"
  sensitive   = true
}

variable "db_password" {
  type        = string
  description = "Database administrator password"
  sensitive   = true
}

provider "aws" {
  access_key = var.aws_access_key
  secret_key = var.aws_secret_key
  region     = "us-east-1"
}

resource "aws_db_instance" "main" {
  password = var.db_password
}
```

### Attribut `sensitive`

```hcl
variable "api_key" {
  type      = string
  sensitive = true  # Masque la valeur dans les logs et outputs
}

output "database_endpoint" {
  value     = aws_db_instance.main.endpoint
  sensitive = false  # Valeur visible
}

output "database_password" {
  value     = var.db_password
  sensitive = true   # Valeur masquée dans terraform output
}
```

**Effet de `sensitive = true`** :
- Valeur masquée par `<sensitive>` dans les logs
- Non affichée dans `terraform plan` ou `terraform apply`
- Masquée dans `terraform output` sauf avec `-raw`

### Exemples Pratiques de Sécurisation

```bash
# Script de déploiement sécurisé
#!/bin/bash

# Chargement des secrets depuis un vault ou fichier sécurisé
export TF_VAR_database_password=$(vault kv get -field=password secret/myapp/db)
export TF_VAR_api_secret_key=$(vault kv get -field=secret secret/myapp/api)

# Variables non sensibles
export TF_VAR_environment="production"
export TF_VAR_region="us-west-2"

# Exécution Terraform
terraform plan
terraform apply -auto-approve

# Nettoyage des variables sensibles
unset TF_VAR_database_password
unset TF_VAR_api_secret_key
```

## Bonnes Pratiques HCL

### Organisation du Code
```hcl
# 1. Blocks terraform et required_providers en premier
terraform {
  required_version = ">= 1.0"
  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

# 2. Configuration des providers
provider "aws" {
  region = var.aws_region
}

# 3. Data sources
data "aws_availability_zones" "available" {
  state = "available"
}

# 4. Locals
locals {
  environment = terraform.workspace
}

# 5. Resources (groupées logiquement)
resource "aws_vpc" "main" {
  # Configuration VPC
}

resource "aws_subnet" "public" {
  # Configuration subnet
}

# 6. Outputs à la fin
output "vpc_id" {
  value = aws_vpc.main.id
}
```

### Formatage et Style
```hcl
# Alignement des = pour la lisibilité
resource "aws_instance" "web" {
  ami                    = data.aws_ami.ubuntu.id
  instance_type          = var.instance_type
  key_name              = var.key_pair_name
  vpc_security_group_ids = [aws_security_group.web.id]
  
  # Espacement entre les sections logiques
  user_data = base64encode(
    templatefile("${path.module}/user-data.sh", {
      environment = var.environment
      app_name   = var.app_name
    })
  )
  
  # Blocks imbriqués avec indentation claire
  ebs_block_device {
    device_name = "/dev/sda1"
    volume_size = 20
    volume_type = "gp3"
  }
}
```