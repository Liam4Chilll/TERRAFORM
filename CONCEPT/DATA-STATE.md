Le **State** (état) de Terraform est un fichier de mapping qui maintient la correspondance entre la configuration déclaré dans le fichier et l'infra déployée.
Les états des données constituent la source de vérité de Terraform pour connaître l'état actuel des ressources gérées.

## Fichier d'État - terraform.tfstate

### Caractéristiques
- **Nom par défaut** : `terraform.tfstate`
- **Format** : JSON structuré
- **Contenu** : Métadonnées complètes de toutes les ressources déployées
- **Fonction** : Référentiel de l'état actuel de l'infrastructure

### Structure du Fichier
```json
{
  "version": 4,
  "terraform_version": "1.5.0",
  "serial": 15,
  "lineage": "12345678-1234-1234-1234-123456789012",
  "outputs": {},
  "resources": [
    {
      "mode": "managed",
      "type": "aws_instance",
      "name": "web_server",
      "provider": "provider[\"registry.terraform.io/hashicorp/aws\"]",
      "instances": [
        {
          "schema_version": 1,
          "attributes": {
            "id": "i-0123456789abcdef0",
            "ami": "ami-0abcdef1234567890",
            "instance_type": "t2.micro",
            "public_ip": "203.0.113.123",
            "private_ip": "10.0.1.100"
          }
        }
      ]
    }
  ]
}
```

## Commandes CLI de Gestion d'État

### Inspection de l'État

#### Lister toutes les ressources
```bash
terraform state list
```
**Sortie exemple** :
```
aws_instance.web_server
aws_security_group.web_sg
aws_s3_bucket.data_storage
data.aws_ami.ubuntu
```

#### Consulter une ressource spécifique
```bash
terraform state show <ADDRESS>
```
**Exemple** :
```bash
terraform state show aws_instance.web_server
```
**Sortie détaillée** :
```
# aws_instance.web_server:
resource "aws_instance" "web_server" {
    ami                                  = "ami-0abcdef1234567890"
    instance_type                       = "t2.micro"
    id                                   = "i-0123456789abcdef0"
    public_ip                           = "203.0.113.123"
    private_ip                          = "10.0.1.100"
    availability_zone                   = "us-west-2a"
    vpc_security_group_ids              = ["sg-0123456789abcdef0"]
    
    tags = {
        "Environment" = "production"
        "Name"        = "WebServer"
    }
}
```

### Manipulation de l'État

#### Déplacer un objet dans l'état
```bash
terraform state mv <SOURCE> <DESTINATION>
```

**Exemples pratiques** :
```bash
# Renommer une ressource
terraform state mv aws_instance.old_name aws_instance.new_name

# Déplacer vers un module
terraform state mv aws_instance.web_server module.web.aws_instance.web_server

# Déplacer depuis un module
terraform state mv module.web.aws_instance.web_server aws_instance.web_server

# Renommer avec index
terraform state mv aws_instance.web_server[0] aws_instance.web_server_primary
```

#### Supprimer une ressource de l'état
```bash
terraform state rm <ADDRESS>
```

**Exemples** :
```bash
# Supprimer une ressource unique
terraform state rm aws_instance.web_server

# Supprimer toutes les instances d'une ressource avec count
terraform state rm aws_instance.web_servers

# Supprimer avec index spécifique
terraform state rm aws_instance.web_servers[1]

# Supprimer un module complet
terraform state rm module.web

# Supprimer plusieurs ressources
terraform state rm aws_instance.web aws_security_group.web_sg
```

**⚠️ Important** : Cette commande supprime uniquement la ressource de l'état Terraform, **pas de l'infrastructure réelle**.

### Gestion Avancée de l'État

#### Afficher l'état complet
```bash
terraform show
```
Affiche l'état actuel dans un format lisible par l'humain.

#### Exporter l'état en JSON
```bash
terraform show -json
```
Sortie complète de l'état au format JSON pour traitement automatique.

#### Télécharger l'état distant
```bash
terraform state pull
```
Récupère l'état depuis le backend distant et l'affiche.

#### Uploader l'état local
```bash
terraform state push <ÉTAT_LOCAL>
```
**⚠️ Attention** : Commande dangereuse qui peut écraser l'état distant.

#### Remplacer l'état (force unlock)
```bash
terraform force-unlock <LOCK_ID>
```
Force le déverrouillage d'un état bloqué en cas de problème.

## Backends d'état

### État Local (par défaut)
```hcl
# Configuration implicite - fichier local
# Fichier créé automatiquement : terraform.tfstate
```

### État distant (recommandé)
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"
    encrypt        = true
  }
}
```

**Autres backends populaires** :
- **Azure Storage** : `azurerm`
- **Google Cloud Storage** : `gcs`
- **Consul** : `consul`
- **HTTP** : `http`
- **Terraform Cloud** : `remote`

## Import de Ressources Existantes

### Importer une ressource dans l'état
```bash
terraform import <ADDRESS> <ID_RÉEL>
```

**Exemples** :
```bash
# Importer une instance EC2 existante
terraform import aws_instance.web_server i-0123456789abcdef0

# Importer un bucket S3
terraform import aws_s3_bucket.data my-existing-bucket

# Importer avec module
terraform import module.web.aws_instance.server i-0123456789abcdef0
```

**Processus d'import** :
1. Créer la configuration Terraform pour la ressource
2. Exécuter la commande import
3. Lancer `terraform plan` pour vérifier la cohérence
4. Ajuster la configuration si nécessaire

```hcl
# 1. Configuration pour ressource à importer
resource "aws_instance" "existing_server" {
  # Configuration minimale requise
  ami           = "ami-0abcdef1234567890"
  instance_type = "t2.micro"
  
  # Les autres attributs seront ajustés après import
}
```

```bash
# 2. Import de la ressource existante
terraform import aws_instance.existing_server i-0123456789abcdef0

# 3. Vérification et ajustement
terraform plan
```

## Verrouillage d'état (State Locking)

### Mécanisme de protection
```hcl
terraform {
  backend "s3" {
    bucket         = "my-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    dynamodb_table = "terraform-state-lock"  # Table pour le verrouillage
  }
}
```

### Gestion des verrous
```bash
# Vérifier l'état du verrou
terraform force-unlock -help

# Forcer le déverrouillage (en cas de blocage)
terraform force-unlock 12345678-1234-1234-1234-123456789012

# Désactiver le verrouillage temporairement
terraform apply -lock=false
```

## Workspaces et état

### Gestion multi-evironnements
```bash
# Créer un workspace
terraform workspace new production

# Lister les workspaces
terraform workspace list

# Changer de workspace
terraform workspace select staging

# Afficher le workspace actuel
terraform workspace show
```

**Structure d'état avec workspaces** :
```
terraform.tfstate.d/
├── production/
│   └── terraform.tfstate
├── staging/
│   └── terraform.tfstate
└── development/
    └── terraform.tfstate
```

## Drift detection et refresh

### Détection des dérives
```bash
# Synchroniser l'état avec l'infrastructure réelle
terraform refresh

# Plan avec refresh automatique
terraform plan -refresh-only

# Appliquer uniquement les changements de refresh
terraform apply -refresh-only
```

### Exemple de drift
```bash
# Situation : Une instance a été modifiée manuellement
terraform plan
```
**Sortie** :
```
# aws_instance.web_server will be updated in-place
~ resource "aws_instance" "web_server" {
    id            = "i-0123456789abcdef0"
  ~ instance_type = "t3.micro" -> "t2.micro"  # Changement détecté
    # (autres attributs inchangés)
}
```

## Bonnes pratiques

### Sécurité
```hcl
terraform {
  backend "s3" {
    bucket         = "secure-terraform-state"
    key            = "prod/terraform.tfstate"
    region         = "us-west-2"
    encrypt        = true                    # Chiffrement obligatoire
    dynamodb_table = "terraform-state-lock"
    
    # Versioning du bucket S3 recommandé
    versioning     = true
  }
}
```

### Isolation des environnements
```bash
# Structure recommandée
environments/
├── production/
│   ├── main.tf
│   ├── variables.tf
│   └── backend.tf    # Backend S3 spécifique prod
├── staging/
│   ├── main.tf
│   ├── variables.tf
│   └── backend.tf    # Backend S3 spécifique staging
└── development/
    ├── main.tf
    ├── variables.tf
    └── backend.tf    # Backend local ou S3 dev
```

### Sauvegarde et versioning
```bash
# Sauvegarde manuelle avant modifications importantes
cp terraform.tfstate terraform.tfstate.backup.$(date +%Y%m%d_%H%M%S)

# Avec backend S3, activer le versioning
aws s3api put-bucket-versioning \
  --bucket my-terraform-state \
  --versioning-configuration Status=Enabled
```

### Troubleshooting 
```bash
# 1. Créer une sauvegarde de l'état actuel
terraform state pull > terraform.tfstate.backup

# 2. Restaurer depuis une sauvegarde
terraform state push terraform.tfstate.backup

# 3. Récupérer l'état depuis le backend
terraform state pull > current_state.json

# 4. Réinitialiser l'état (DANGEREUX)
rm terraform.tfstate*
terraform init
# Puis réimporter toutes les ressources
```

## Cas d'usage avancés

### Migration d'État
```bash
# Changer de backend
# 1. Modifier la configuration backend
# 2. Réinitialiser avec migration
terraform init -migrate-state

# Réinitialiser sans backend
terraform init -backend=false
```

### État partagé en Équipe
```hcl
terraform {
  backend "remote" {
    organization = "my-company"
    
    workspaces {
      name = "production-app"
    }
  }
}
```

### Ressources sensibles dans l'état
```bash
# L'état peut contenir des données sensibles
# Chiffrement obligatoire pour les backends distants
# Contrôle d'accès strict aux fichiers d'état

# Vérifier le contenu sensible
terraform show | grep -i password
```