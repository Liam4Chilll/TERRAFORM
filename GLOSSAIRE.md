# Dictionnaire Terraform - Lexique Complet

## A

**Apply**  
Action d'exécuter un plan Terraform pour créer, modifier ou détruire des ressources selon la configuration

**Argument**  
Paramètre assigné à une ressource ou module pour définir sa configuration spécifique

**Attribute**  
Propriété d'une ressource accessible en lecture seule après sa création

**Auto-approve**  
Flag permettant d'exécuter apply ou destroy sans confirmation interactive

## B

**Backend**  
Système de stockage pour l'état Terraform (local, S3, Consul, etc.)

**Block**  
Structure de configuration délimitée par des accolades contenant des arguments

**Built-in Function**  
Fonction prédéfinie dans Terraform pour manipuler des données (length, merge, etc.)

## C

**Configuration**  
Ensemble de fichiers .tf décrivant l'infrastructure désirée

**Console**  
Interface interactive pour tester les expressions et fonctions Terraform

**Count**  
Meta-argument pour créer plusieurs instances d'une ressource

**Cycle**  
Dépendance circulaire entre ressources causant une erreur de planification

## D

**Data Source**  
Ressource en lecture seule pour récupérer des informations d'infrastructure existante

**Dependency**  
Relation entre ressources définissant l'ordre de création/destruction

**Destroy**  
Action de suppression de toutes les ressources gérées par Terraform

**Drift**  
Écart entre l'état Terraform et l'infrastructure réelle

## E

**Expression**  
Combinaison de valeurs, opérateurs et fonctions retournant une valeur

**Export**  
Action de rendre visible une valeur via un output

## F

**For_each**  
Meta-argument pour créer plusieurs instances basées sur un map ou set

**Function**  
Opération prédéfinie pour transformer ou calculer des valeurs

## G

**Graph**  
Représentation des dépendances entre ressources sous forme de graphe dirigé

## H

**HCL (HashiCorp Configuration Language)**  
Langage de configuration utilisé par Terraform

**Hook**  
Point d'exécution permettant d'insérer des actions personnalisées

## I

**Import**  
Action d'intégrer une ressource existante dans l'état Terraform

**Implicit Dependency**  
Dépendance automatiquement détectée via les références entre ressources

**Input Variable**  
Paramètre d'entrée permettant de personnaliser une configuration

**Interpolation**  
Insertion de valeurs dynamiques dans des chaînes de caractères

## J

**JSON**  
Format alternatif pour écrire les configurations Terraform (.tf.json)

## L

**Lifecycle**  
Règles personnalisées pour contrôler le comportement des ressources

**Local Value**  
Variable calculée localement réutilisable dans la configuration

**Lock**  
Mécanisme empêchant les exécutions concurrentes sur le même état

## M

**Module**  
Package réutilisable de configuration Terraform

**Meta-argument**  
Argument spécial disponible pour tous les types de ressources

## N

**Null Resource**  
Ressource virtuelle pour exécuter des actions sans créer d'infrastructure

## O

**Output**  
Valeur exposée par une configuration Terraform

**Override**  
Mécanisme pour modifier des configurations sans les éditer

## P

**Plan**  
Aperçu des actions que Terraform va exécuter avant l'application

**Provider**  
Plugin permettant à Terraform d'interagir avec une API spécifique

**Provisioner**  
Mécanisme pour exécuter des scripts après la création d'une ressource

## R

**Refresh**  
Action de synchronisation de l'état avec l'infrastructure réelle

**Remote State**  
État Terraform stocké sur un backend distant

**Resource**  
Composant d'infrastructure géré par Terraform

**Root Module**  
Module principal contenant la configuration de base

## S

**State**  
Fichier de mapping entre configuration et infrastructure réelle

**State File**  
Fichier JSON contenant l'état actuel de l'infrastructure

**State Lock**  
Verrouillage empêchant les modifications concurrentes de l'état

## T

**Taint**  
Marquage d'une ressource pour forcer sa recréation

**Target**  
Limitation d'une opération à des ressources spécifiques

**Terraform Block**  
Configuration des paramètres globaux (version, backend, providers)

## U

**Untaint**  
Suppression du marquage de recréation d'une ressource

## V

**Variable**  
Paramètre d'entrée pour personnaliser une configuration

**Validation**  
Vérification de la syntaxe et cohérence des fichiers de configuration

**Version Constraint**  
Spécification des versions acceptables pour providers et modules

## W

**Workspace**  
Environnement isolé permettant de gérer plusieurs états

**Working Directory**  
Répertoire contenant les fichiers de configuration Terraform

## Concepts Avancés

**Dynamic Block**  
Bloc généré conditionnellement basé sur des données complexes

**Sensitive**  
Marquage pour masquer des valeurs dans les logs et outputs

**Moved Block**  
Déclaration pour gérer le renommage de ressources sans destruction

**Precondition/Postcondition**  
Vérifications personnalisées avant et après les opérations sur ressources

**Check Block**  
Validation continue de l'infrastructure sans impact sur l'état

**Import Block**  
Déclaration d'import de ressources existantes dans la configuration

**Replaced**  
Stratégie de mise à jour forçant la recréation d'une ressource

**Depends_on**  
Déclaration explicite de dépendance entre ressources

**Ignore_changes**  
Directive pour ignorer les modifications sur certains attributs

**Create_before_destroy**  
Stratégie de recréation créant la nouvelle ressource avant détruire l'ancienne

**Prevent_destroy**  
Protection empêchant la suppression accidentelle d'une ressource

**Replace_triggered_by**  
Déclenchement de recréation basé sur les changements d'autres ressources

## Types de Données

**String**  
Chaîne de caractères délimitée par des guillemets

**Number**  
Valeur numérique (entier ou décimal)

**Bool**  
Valeur booléenne (true/false)

**List**  
Collection ordonnée de valeurs du même type

**Set**  
Collection non ordonnée de valeurs uniques

**Map**  
Collection de paires clé-valeur

**Object**  
Structure complexe avec des attributs typés

**Tuple**  
Liste avec des types différents pour chaque élément

**Any**  
Type acceptant n'importe quelle valeur

**Null**  
Absence de valeur

## Opérateurs

**Arithmetic**  
+, -, *, /, % pour les opérations mathématiques

**Comparison**  
==, !=, <, >, <=, >= pour comparer des valeurs

**Logical**  
&&, ||, ! pour les opérations booléennes

**Conditional**  
condition ? true_val : false_val pour les expressions conditionnelles

## Fonctions Courantes

**length()**  
Retourne la taille d'une liste, map ou chaîne

**merge()**  
Combine plusieurs maps en un seul

**join()**  
Concatene les éléments d'une liste avec un séparateur

**split()**  
Divise une chaîne en liste selon un délimiteur

**lookup()**  
Recherche une valeur dans un map avec valeur par défaut

**element()**  
Retourne l'élément à un index donné d'une liste

**keys()**  
Retourne les clés d'un map sous forme de liste

**values()**  
Retourne les valeurs d'un map sous forme de liste

**contains()**  
Vérifie si une liste contient une valeur spécifique

**formatdate()**  
Formate une date selon un pattern spécifié

**base64encode()**  
Encode une chaîne en Base64

**jsonencode()**  
Convertit une valeur en JSON

**templatefile()**  
Traite un fichier template avec des variables

**file()**  
Lit le contenu d'un fichier local

**pathexpand()**  
Résout les chemins relatifs et variables d'environnement

## États et Cycles

**Plan Phase**  
Étape de calcul des changements nécessaires

**Apply Phase**  
Étape d'exécution des changements planifiés

**Destroy Phase**  
Étape de suppression des ressources

**Refresh Phase**  
Étape de synchronisation avec l'infrastructure réelle

**Graph Phase**  
Étape de construction du graphe de dépendances

**Validate Phase**  
Étape de vérification de la configuration

## Erreurs Communes

**Cycle Error**  
Dépendance circulaire entre ressources

**Lock Error**  
Impossibilité d'acquérir le verrou sur l'état

**Authentication Error**  
Échec d'authentification avec le provider

**Resource Not Found**  
Ressource supprimée manuellement mais présente dans l'état

**Version Conflict**  
Incompatibilité entre versions de Terraform et providers

**State Corruption**  
Fichier d'état endommagé ou incohérent

**Drift Detection**  
Détection de modifications externes à Terraform

**Plan Inconsistency**  
Différence entre plan et réalité lors de l'application