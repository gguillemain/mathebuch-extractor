# mathebuch-extractor
extraire des exercices d'un fichier latex pour creer un json
# Mathebuch - Extraction LaTeX avec Claude

Ce projet fournit des outils pour extraire des exercices et leçons mathématiques depuis des fichiers LaTeX, les enrichir via l'API Claude, puis les importer dans une base de données MySQL pour votre application Mathebuch.

## Caractéristiques principales

- **Extraction intelligente** : Analyse les fichiers LaTeX pour identifier leçons et exercices
- **Enrichissement IA** : Utilise l'API Claude pour analyser le contenu et générer des métadonnées de qualité
- **Structure avec 3 compétences** : Respecte le modèle avec 3 compétences distinctes (competence_1, competence_2, competence_3)
- **Importation MySQL** : Stocke les données structurées dans une base de données relationnelle
- **Interface CLI** : Simplifie l'utilisation via des commandes intuitives

## Prérequis

- Python 3.8+
- MySQL 8.0+
- Une clé API Claude (Anthropic)
- Bibliothèques Python : `mysql-connector-python`, `anthropic`, `python-dotenv`

## Installation

1. Clonez ce dépôt :

```bash
git clone https://github.com/votre-utilisateur/mathebuch-extractor.git
cd mathebuch-extractor
```

2. Installez les dépendances :

```bash
pip install mysql-connector-python anthropic python-dotenv
```

3. Initialisez la base de données avec le schéma fourni :

```bash
mysql -u votre_utilisateur -p < mysql-schema-competences.sql
```

4. Créez un fichier `.env` avec votre clé API Claude :

```
ANTHROPIC_API_KEY=votre_clé_api_claude
```

## Utilisation

### Extraction et enrichissement via Claude

Pour extraire une leçon et ses exercices d'un fichier LaTeX et les enrichir avec Claude :

```bash
python latex_claude_extractor.py chemin/vers/votre/fichier.tex --grade 6eme --output ./output
```

Options :
- `--grade` : Spécifie le niveau scolaire (6eme, 5eme, 4eme, 3eme)
- `--output` : Répertoire où sauvegarder les fichiers JSON
- `--api-key` : Clé API Claude (sinon utilise la variable d'environnement)
- `--skip-enrich` : Extraction basique sans enrichissement via Claude

### Importation des données dans MySQL

Pour importer les données extraites dans votre base de données :

```bash
python json_to_mysql_competences.py ./output/extracted_content.json --host localhost --user root --database mathebuch_db --ask-password
```

Options :
- `--host` : Hôte MySQL (défaut : localhost)
- `--user` : Nom d'utilisateur MySQL (défaut : root)
- `--database` : Nom de la base de données (défaut : mathebuch_db)
- `--ask-password` : Demande le mot de passe de manière interactive
- `--password` : Mot de passe MySQL (non recommandé, préférez --ask-password)

### Commande complète (extraction + importation)

Pour effectuer l'ensemble du processus en une seule commande :

```bash
python extract_import_cli.py chemin/vers/votre/fichier.tex --grade 6eme --output-dir ./output --ask-password
```

Cette commande extrait les données, les enrichit via Claude et les importe dans la base de données MySQL.

## Structure des données

### Exercice JSON

```json
{
  "id": "exo_6eme_1",
  "title": "Exercice 1",
  "description": "Exercice sur les fractions",
  "grade_level": "6eme",
  "domaine": "Nombres",
  "competence_1": "calculer",
  "competence_2": "raisonner",
  "competence_3": null,
  "sub_topic": "Fractions",
  "difficulty": 2,
  "latex_content": "...",
  "numero": "1",
  "page": 42,
  "answers": { "q1": "3/4", "q2": "5/6" },
  "hints": { "q1": "Simplifie d'abord" },
  "temps_estime": 10,
  "mots_cles": ["fraction", "simplification"],
  "has_illustration": false,
  "origine": "initial",
  "lesson_id": "lesson_abc123"
}
```

### Leçon JSON

```json
{
  "id": "lesson_abc123",
  "chapter_id": "chap_fractions",
  "title": "Les fractions",
  "grade_level": "6eme",
  "domaine": "Nombres",
  "latex_content": "...",
  "html_content": "...",
  "vocabulary": [
    { "term": "fraction", "definition_fr": "rapport entre deux nombres", "definition_de": "der Bruch" }
  ],
  "key_concepts": ["simplification", "numérateur"],
  "examples": ["3/4 + 1/2 = 5/4"],
  "related_exercises": ["exo_6eme_1"]
}
```

## Schéma de la base de données

Le schéma MySQL est conçu pour stocker les données avec une structure claire :

- `chapitres` : Contient les informations sur les chapitres
- `lecons` : Stocke les leçons avec leur contenu LaTeX et HTML
- `vocabulaire`, `concepts_cles`, `exemples` : Tables liées aux leçons
- `exercices` : Stocke les exercices avec trois champs de compétences distincts
- `mots_cles` : Mots-clés pouvant être associés aux exercices
- `indices`, `reponses` : Tables pour stocker les indices et réponses des exercices

## Analyse des coûts API Claude

Pour un fichier LaTeX typique contenant une leçon et 10-15 exercices :

- **Claude 3 Haiku** : ~ 0,01-0,02€ par fichier
- **Claude 3 Sonnet** : ~ 0,10-0,15€ par fichier
- **Claude 3 Opus** : ~ 0,50-0,75€ par fichier

Pour un manuel complet avec 50 leçons :
- Avec Claude 3 Haiku : ~ 0,5-1€
- Avec Claude 3 Sonnet : ~ 5-7,5€
- Avec Claude 3 Opus : ~ 25-35€

## Avantages de l'utilisation de Claude

1. **Qualité des métadonnées** : Claude comprend le contexte mathématique et peut identifier correctement les compétences, domaines et niveaux de difficulté
2. **Flexibilité** : Adaptation automatique aux variations de structure dans les documents LaTeX
3. **Gain de temps** : Automatisation du travail d'annotation qui serait très chronophage manuellement
4. **Enrichissement sémantique** : Extraction de vocabulaire, concepts clés et exemples que des expressions régulières ne pourraient pas identifier

## Personnalisation

Vous pouvez personnaliser les prompts envoyés à Claude pour améliorer les résultats :

1. Dans le fichier `latex_claude_extractor.py`, modifiez les méthodes `_create_lesson_prompt` et `_create_exercise_prompt`
2. Ajoutez des exemples spécifiques pour améliorer la compréhension de Claude
3. Enrichissez les instructions pour des domaines mathématiques particuliers

## Dépannage

### L'API Claude renvoie une erreur

- Vérifiez que votre clé API est valide et correctement configurée
- Assurez-vous que vous n'avez pas atteint vos limites d'utilisation
- Vérifiez que les prompts ne sont pas trop longs (limite de tokens)

### Problèmes d'importation MySQL

- Vérifiez les identifiants de connexion et les permissions
- Assurez-vous que la base de données existe et a été correctement initialisée
- Vérifiez que le schéma correspond exactement à celui défini dans `mysql-schema-competences.sql`

## Licence

Ce projet est destiné à l'usage interne pour votre application Mathebuch.

## Contributeurs

- [Votre nom]
- [Contributeur 2]

## Remerciements

- Anthropic pour l'API Claude
- L'équipe du projet Mathebuch pour les spécifications initiales"
