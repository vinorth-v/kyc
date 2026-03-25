# KYC Document Processing

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Python 3.10+](https://img.shields.io/badge/python-3.10+-blue.svg)](https://www.python.org/downloads/)

Traitement automatique de documents KYC (Know Your Customer) avec des LLM multimodaux (Gemini, GPT-4V, Claude).

## Pourquoi ce projet ?

### Le probleme avec le deep learning classique

Pour creer un systeme de classification et extraction de documents KYC avec deep learning classique:

- **Plusieurs mois de developpement**: collecte de donnees, annotation, entrainement de modeles CNN, post-processing OCR
- **Couts eleves**: infrastructure GPU, equipe ML, data labelers
- **Resultats limites**: ne fonctionne que sur les formats appris, necessite re-entrainement pour chaque variation

### La solution avec les LLM multimodaux

- **Quelques heures de developpement**: prompts + schemas Pydantic + regles metier
- **Couts minimes**: API calls, zero infrastructure d'entrainement
- **Resultats superieurs**: fonctionne out-of-the-box sur nouveaux formats, robuste aux variations

## Fonctionnalites

### Documents traites

1. **Pieces d'identite**
   - Carte Nationale d'Identite (CNI)
   - Passeport
   - Permis de conduire (avec detection des categories cochees)

2. **Justificatifs de domicile**
   - Factures (electricite, gaz, eau, internet, telephone)
   - Quittances de loyer
   - Taxes
   - Attestations d'assurance

3. **Coordonnees bancaires**
   - RIB/IBAN avec validation checksum modulo 97

### Regles metier

- Validation de coherence entre documents (nom, prenom)
- Verification des dates d'expiration
- Controle de l'anciennete (justificatif < 3 mois)
- Validation technique IBAN (checksum)
- Detection visuelle des cases cochees (permis de conduire)

## Installation

### Prerequis

- Python 3.10+
- Acces Google Cloud avec Vertex AI active
- [uv](https://docs.astral.sh/uv/) (gestionnaire de packages)
- [just](https://github.com/casey/just) (task runner, optionnel)

### Setup

```bash
# Cloner le projet
git clone https://github.com/votre-username/kyc-document-processing.git
cd kyc-document-processing

# Installer les dependances
uv sync

# Configuration
cp .env.example .env
# Editer .env avec vos credentials

# Exporter les credentials Google Cloud
export GOOGLE_APPLICATION_CREDENTIALS="path/to/your/credentials.json"
```

## Utilisation

### Document individuel

```bash
uv run python src/main.py path/to/document.pdf
```

### Dossier complet

```bash
uv run python src/main.py --folder path/to/folder/
```

### En Python

```python
from chains.llm_chain import KYCDocumentChain
from pipeline import KYCPipeline

# Un document
chain = KYCDocumentChain()
result = chain.process_document("document.jpg")

# Dossier complet
pipeline = KYCPipeline()
dossier = pipeline.process_folder("dossier_client/")
```

### Commandes just (optionnel)

```bash
just --list  # Affiche toutes les commandes disponibles
just check   # Formate, lint et tests
just test    # Lance les tests
```

## Architecture

```
kyc-document-processing/
├── src/
│   ├── chains/
│   │   ├── schemas/
│   │   │   └── kyc_schemas.py      # Schemas Pydantic pour chaque doc
│   │   ├── configuration.py         # Config Google Cloud / Vertex AI
│   │   ├── llm_chain.py            # Chain LLM principale
│   │   └── prompts.py              # Prompts pour classification/extraction
│   ├── utils/
│   │   └── config.py               # Utilitaires de configuration
│   ├── pipeline.py                 # Pipeline multi-documents
│   └── main.py                     # Point d'entree
├── tests/
│   └── test_schemas.py             # Tests unitaires
└── config/
    └── config.json                 # Configuration du projet
```

## Exemples de code

### Classification automatique

```python
chain = KYCDocumentChain()
result = chain.classify_document("document.jpg")
# -> "carte_identite" avec 98% de confiance
```

### Extraction structuree

```python
cni = chain.extract_cni("cni.jpg")
print(f"{cni.prenom} {cni.nom}")
print(f"Valide: {cni.est_valide}")
```

### Detection des cases cochees (permis)

```python
permis = chain.extract_permis("permis.jpg")
print(permis.categories)
# -> ["B", "A2"]
```

### Validation IBAN

```python
rib = chain.extract_rib("rib.jpg")
print(f"IBAN: {rib.iban}")
print(f"Checksum valide: {rib.iban_valide}")
```

## Tests

```bash
uv run pytest tests/ -v
```

## Contribuer

Les contributions sont les bienvenues ! Consultez [CONTRIBUTING.md](CONTRIBUTING.md) pour les details.

## Licence

Ce projet est sous licence MIT. Voir [LICENSE](LICENSE) pour plus de details.

## Ressources

- [Vertex AI Gemini](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/overview)
- [Pydantic](https://docs.pydantic.dev/)
- [Google Document AI](https://cloud.google.com/document-ai)
