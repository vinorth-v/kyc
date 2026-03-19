# 🏦 Démonstrateur KYC - Conférence

## 🎯 Objectif

Démonstration live de la **révolution apportée par les LLM multimodaux** dans le traitement de documents KYC (Know Your Customer).

### Le problème avant

Pour créer un système de classification et extraction de documents KYC avec deep learning classique:

❌ **Plusieurs mois de développement**
- Collecter et annoter des milliers d'images de documents
- Créer des bounding boxes pour chaque champ à extraire
- Entraîner des modèles CNN pour la classification
- Entraîner des modèles de détection d'objets pour localiser les champs
- Créer un classificateur binaire pour les cases à cocher (cochée/non cochée)
- Développer du post-processing OCR complexe
- Coder des règles spécifiques pour chaque type de document
- Maintenance cauchemardesque à chaque nouveau format

❌ **Coûts élevés**
- Infrastructure GPU pour l'entraînement
- Équipe de 3-5 ML engineers
- Data labelers pour annotation
- Storage pour les datasets

❌ **Résultats limités**
- Ne fonctionne que sur les formats appris
- Nécessite ré-entraînement pour chaque variation
- Peu robuste aux changements de mise en page

### La solution maintenant

Avec les LLM multimodaux (Gemini, GPT-4V, Claude):

✅ **Quelques heures de développement**
- Rédiger des prompts clairs
- Définir des schémas Pydantic pour la structure
- Coder les règles métier en Python

✅ **Coûts minimes**
- API calls (quelques centimes par document)
- Zéro infrastructure d'entraînement
- Une seule personne peut tout développer

✅ **Résultats supérieurs**
- Fonctionne out-of-the-box sur nouveaux formats
- Comprend le contexte et les variations
- Robuste aux changements de mise en page

## 📋 Use Case: KYC Bancaire

### Documents traités

1. **Pièces d'identité**
   - Carte Nationale d'Identité (CNI)
   - Passeport
   - Permis de conduire
   
2. **Justificatifs de domicile**
   - Factures (électricité, gaz, eau, internet, téléphone)
   - Quittances de loyer
   - Taxes
   - Attestations d'assurance

3. **Coordonnées bancaires**
   - RIB/IBAN avec validation checksum modulo 97

### Règles métier

✓ Validation de cohérence entre documents (nom, prénom)  
✓ Vérification des dates d'expiration  
✓ Contrôle de l'ancienneté (justificatif < 3 mois)  
✓ Validation technique IBAN (checksum)  
✓ Détection visuelle des cases cochées (permis de conduire)

## 🚀 Architecture

```
demonstrateur_KYC_grosse_conf/
├── src/
│   ├── chains/
│   │   ├── schemas/
│   │   │   └── kyc_schemas.py      # Schémas Pydantic pour chaque doc
│   │   ├── configuration.py         # Config Google Cloud / Vertex AI
│   │   ├── llm_chain.py            # Chain LLM principale
│   │   └── prompts.py              # Prompts pour classification/extraction
│   ├── utils/
│   │   └── config.py               # Utilitaires de configuration
│   ├── pipeline.py                 # Pipeline multi-documents
│   └── main.py                     # Point d'entrée / démo
├── tests/
│   └── test_schemas.py             # Tests unitaires
├── config/
│   └── config.json                 # Configuration du projet
└── examples/                       # Documents d'exemple pour la démo
```

## 🎪 Points à démontrer en conférence

### 1. Classification automatique

```python
# Avant: modèle CNN entraîné sur 10k+ images
# Maintenant: un prompt!

chain = KYCDocumentChain()
result = chain.classify_document("document.jpg")
# → "carte_identite" avec 98% de confiance
```

### 2. Extraction structurée

```python
# Le LLM retourne directement un objet Pydantic validé
cni = chain.extract_cni("cni.jpg")
print(f"{cni.prenom} {cni.nom}")
print(f"Valide: {cni.est_valide}")
```

### 3. ⭐ Cas des cases à cocher (KILLER FEATURE)

**Permis de conduire: quelles catégories sont cochées?**

```python
permis = chain.extract_permis("permis.jpg")
print(permis.categories)
# → ["B", "A2"]
```

**Avant:**
1. Créer un modèle de détection pour chaque case (AM, A1, A2, A, B, BE, C, etc.)
2. Annoter des milliers d'images avec bounding boxes précises
3. Entraîner un classificateur binaire (coché/pas coché)
4. Gérer les cas ambigus (case partiellement cochée, croix vs coche)

**Maintenant:**
> "Quelles catégories sont cochées?"

C'EST TOUT! 🤯

### 4. Validation IBAN

```python
rib = chain.extract_rib("rib.jpg")
print(f"IBAN: {rib.iban}")
print(f"Checksum valide: {rib.iban_valide}")
```

Le LLM extrait, Python valide avec modulo 97.

### 5. Cohérence multi-documents

```python
dossier = DossierKYC(
    piece_identite=cni,
    justificatif_domicile=justif,
    rib=rib
)

dossier.valider_coherence()
# Vérifie: même nom sur tous les docs, dates valides, etc.
```

## 💻 Installation

### Prérequis

- Python 3.10+
- Accès Google Cloud avec Vertex AI activé
- [uv](https://docs.astral.sh/uv/) (gestionnaire de packages ultra-rapide)
- [just](https://github.com/casey/just) (task runner, optionnel)

### Setup

```bash
# Cloner le projet
git clone <repo-url>
cd demonstrateur_KYC_grosse_conf

# Installer les dépendances (uv crée automatiquement le .venv)
just sync
# ou
uv sync

# Configuration Google Cloud
cp .env.example .env
# Éditer .env avec vos credentials

# Exporter les credentials
export GOOGLE_APPLICATION_CREDENTIALS="path/to/your/credentials.json"  # Linux/Mac
# ou
$env:GOOGLE_APPLICATION_CREDENTIALS="path\to\your\credentials.json"  # Windows PowerShell
```

> **Note**: `just --list` affiche toutes les commandes disponibles.

### Configuration

Éditer `config/config.json`:

```json
{
  "project_id": "votre-project-id",
  "location": "europe-west1",
  "model": "gemini-1.5-pro-002",
  "temperature": 0.0
}
```

## 🎬 Utilisation pour la démo

### Documents individuels

```bash
just cni        # ou: uv run python src/main.py examples/cni.webp
just passeport  # ou: uv run python src/main.py examples/passeport.webp
just permis     # ou: uv run python src/main.py examples/permis_recto_verso.pdf
just edf        # ou: uv run python src/main.py examples/jdom_edf.pdf
just impots     # ou: uv run python src/main.py examples/jdom_impots.pdf
```

### Dossier complet

```bash
just dossier
# ou: uv run python src/main.py --folder examples/
```

### Document personnalisé

```bash
just run path/to/document.pdf
# ou: uv run python src/main.py path/to/document.pdf
```

> `just --list` affiche toutes les commandes disponibles.

### En code Python

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

## 🧪 Tests

```bash
just test   # ou: uv run pytest tests/ -v
just check  # Formatte, lint et tests en une commande
```

## 📊 Métriques de comparaison

| Critère | Deep Learning | LLM Multimodal |
|---------|--------------|----------------|
| **Temps de développement** | 3-6 mois | 1-3 jours |
| **Dataset requis** | 10k+ images annotées | Zéro |
| **Coût setup** | 50-100k€ | ~0€ |
| **Coût par document** | ~0.01€ (infra) | ~0.03€ (API) |
| **Maintenance** | Élevée | Faible |
| **Nouveaux formats** | Ré-entraînement | Adaptation prompt |
| **Précision** | 85-92% | 90-97% |

## 🎓 Messages clés pour la conférence

1. **Paradigm shift**: On passe de "entraîner un modèle" à "poser les bonnes questions"

2. **Démocratisation**: Plus besoin d'une équipe ML avec PhD pour traiter des documents

3. **Time-to-market**: De 6 mois à quelques jours

4. **Cas d'usage killer**: Tout ce qui nécessitait de la vision + compréhension contextuelle
   - Documents avec mise en page variable
   - Cases à cocher
   - Formats multiples
   - Validation de cohérence

5. **Limites actuelles**: 
   - Coût par document (vs one-time training cost)
   - Latence légèrement supérieure
   - Dépendance à un provider externe

6. **Tendance**: Les modèles deviennent meilleurs et moins chers chaque mois

## 📝 Script de démo

### Introduction (2 min)

"Imaginez: vous devez créer un système pour traiter des documents KYC bancaires. 
CNI, passeports, justificatifs de domicile, RIB.

Il y a 2 ans, vous auriez commencé par:
- Collecter 10 000 images annotées
- Créer des bounding boxes pour chaque champ
- Entraîner un CNN pour la classification
- etc.

6 mois et 100k€ plus tard, vous avez un système qui fonctionne... 
sur les formats que vous avez appris.

Maintenant, regardez..."

### Démo live (8 min)

1. **Classification** (1 min)
   - Montrer plusieurs types de documents
   - Le LLM les identifie instantanément

2. **Extraction CNI** (2 min)
   - Afficher une CNI
   - Extraction complète en JSON
   - Validation de la date d'expiration

3. **⭐ Cases à cocher - permis** (3 min)
   - **LE MOMENT FORT**
   - Afficher un permis avec catégories cochées
   - "Avant: detection model + classificateur binaire + bounding boxes"
   - "Maintenant: regardez"
   - Extraction des catégories
   - "C'est tout!"

4. **Dossier complet** (2 min)
   - Pipeline sur dossier entier
   - Validation de cohérence
   - Rapport final

### Conclusion (2 min)

"De 6 mois à 2 jours. De 100k€ à presque rien.
Meilleure précision. Plus flexible.

C'est ça, la révolution des LLM multimodaux.

Et on n'a pas encore vu le plein potentiel.
Gemini 2.0, GPT-5... ça ne fait que commencer."

## 🔗 Ressources

- [Vertex AI Gemini](https://cloud.google.com/vertex-ai/generative-ai/docs/multimodal/overview)
- [Pydantic](https://docs.pydantic.dev/)
- [Google Document AI](https://cloud.google.com/document-ai) (approche alternative OCR d'abord)

## 📧 Contact

Pour questions sur la démo ou le code.

---

**Note**: Ce projet est un démonstrateur éducatif pour conférences. 
Pour une utilisation en production, ajouter:
- Gestion d'erreurs robuste
- Logging structuré
- Monitoring des coûts API
- Cache des résultats
- Tests de régression visuels
- Pipeline CI/CD
