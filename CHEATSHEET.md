# Terraform Operations Cheatsheet

## Validation et Formatage

```bash
terraform fmt
```
Formater les fichiers de configuration Terraform

```bash
terraform fmt -check
```
Vérifier si les fichiers sont correctement formatés

```bash
terraform validate
```
Valider la syntaxe et cohérence des fichiers de configuration

```bash
terraform validate -json
```
Valider et retourner les erreurs en format JSON

## Initialisation et Plugins

```bash
terraform init
```
Initialiser le répertoire de travail et télécharger les providers

```bash
terraform init -upgrade
```
Mettre à jour les providers vers les dernières versions compatibles

```bash
terraform init -backend-config="bucket=<BUCKET_NAME>"
```
Initialiser avec configuration backend spécifique

```bash
terraform init -migrate-state
```
Migrer l'état vers un nouveau backend

## Planification

```bash
terraform plan
```
Créer un plan d'exécution montrant les changements à apporter

```bash
terraform plan -out=<PLAN_FILE>
```
Sauvegarder le plan dans un fichier

```bash
terraform plan -target=<RESOURCE_TYPE>.<RESOURCE_NAME>
```
Planifier uniquement pour une ressource spécifique

```bash
terraform plan -var="<VARIABLE_NAME>=<VALUE>"
```
Planifier avec une variable spécifique

```bash
terraform plan -var-file="<VAR_FILE>.tfvars"
```
Planifier avec un fichier de variables

```bash
terraform plan -destroy
```
Planifier la destruction de toutes les ressources

## Application

```bash
terraform apply
```
Appliquer les changements pour atteindre l'état désiré

```bash
terraform apply <PLAN_FILE>
```
Appliquer un plan préalablement sauvegardé

```bash
terraform apply -auto-approve
```
Appliquer sans demander de confirmation

```bash
terraform apply -target=<RESOURCE_TYPE>.<RESOURCE_NAME>
```
Appliquer uniquement pour une ressource spécifique

```bash
terraform apply -var="<VARIABLE_NAME>=<VALUE>"
```
Appliquer avec une variable spécifique

```bash
terraform apply -parallelism=<NUMBER>
```
Limiter le nombre d'opérations parallèles

## Gestion de l'État

```bash
terraform show
```
Afficher l'état actuel ou un plan sauvegardé

```bash
terraform show -json
```
Afficher l'état en format JSON

```bash
terraform state list
```
Lister toutes les ressources dans l'état

```bash
terraform state show <RESOURCE_TYPE>.<RESOURCE_NAME>
```
Afficher les détails d'une ressource spécifique

```bash
terraform state mv <SOURCE> <DESTINATION>
```
Déplacer une ressource dans l'état

```bash
terraform state rm <RESOURCE_TYPE>.<RESOURCE_NAME>
```
Supprimer une ressource de l'état sans la détruire

```bash
terraform state pull
```
Télécharger et afficher l'état distant

```bash
terraform state push <STATE_FILE>
```
Uploader un fichier d'état local vers le backend distant

## Import et Refresh

```bash
terraform import <RESOURCE_TYPE>.<RESOURCE_NAME> <RESOURCE_ID>
```
Importer une ressource existante dans l'état Terraform

```bash
terraform refresh
```
Mettre à jour l'état avec l'infrastructure réelle

## Outputs et Variables

```bash
terraform output
```
Afficher toutes les valeurs de sortie

```bash
terraform output <OUTPUT_NAME>
```
Afficher une valeur de sortie spécifique

```bash
terraform output -json
```
Afficher les outputs en format JSON

```bash
terraform console
```
Lancer une console interactive pour tester les expressions

## Workspaces

```bash
terraform workspace list
```
Lister tous les workspaces disponibles

```bash
terraform workspace show
```
Afficher le workspace actuel

```bash
terraform workspace new <WORKSPACE_NAME>
```
Créer et basculer vers un nouveau workspace

```bash
terraform workspace select <WORKSPACE_NAME>
```
Basculer vers un workspace existant

```bash
terraform workspace delete <WORKSPACE_NAME>
```
Supprimer un workspace

## Provisioners et Remote Execution

```bash
terraform taint <RESOURCE_TYPE>.<RESOURCE_NAME>
```
Marquer une ressource pour recréation au prochain apply

```bash
terraform untaint <RESOURCE_TYPE>.<RESOURCE_NAME>
```
Enlever la marque de recréation d'une ressource

```bash
terraform force-unlock <LOCK_ID>
```
Forcer le déverrouillage de l'état

## Debugging et Logs

```bash
TF_LOG=DEBUG terraform plan
```
Activer les logs de débogage pour une commande

```bash
TF_LOG=TRACE terraform apply
```
Activer les logs de trace détaillés

```bash
TF_LOG_PATH=<LOG_FILE> terraform apply
```
Rediriger les logs vers un fichier

## Version et Providers

```bash
terraform version
```
Afficher la version de Terraform et des providers

```bash
terraform providers
```
Lister les providers requis par la configuration

```bash
terraform providers lock
```
Verrouiller les versions des providers

```bash
terraform providers schema -json
```
Afficher le schéma des providers en JSON

## Graph et Documentation

```bash
terraform graph
```
Générer un graphe des dépendances en format DOT

```bash
terraform graph | dot -Tpng > <GRAPH_FILE>.png
```
Générer un graphique visuel des dépendances

## Test et Validation Avancée

```bash
terraform test
```
Exécuter les tests de configuration Terraform

```bash
terraform test -filter=<TEST_NAME>
```
Exécuter un test spécifique

## Remote State et Backend

```bash
terraform init -backend=false
```
Initialiser sans configurer le backend

```bash
terraform init -reconfigure
```
Reconfigurer le backend en ignorant l'état existant

```bash
terraform init -backend-config="<KEY>=<VALUE>"
```
Initialiser avec configuration backend dynamique

## Destruction

```bash
terraform destroy
```
Détruire toutes les ressources gérées par Terraform

```bash
terraform destroy -auto-approve
```
Détruire sans demander de confirmation

```bash
terraform destroy -target=<RESOURCE_TYPE>.<RESOURCE_NAME>
```
Détruire uniquement une ressource spécifique

```bash
terraform destroy -var="<VARIABLE_NAME>=<VALUE>"
```
Détruire avec une variable spécifique

```bash
terraform destroy -var-file="<VAR_FILE>.tfvars"
```
Détruire avec un fichier de variables