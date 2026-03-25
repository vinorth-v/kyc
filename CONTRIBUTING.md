# Contribuer au projet

Merci de votre interet pour ce projet ! Voici comment contribuer.

## Signaler un bug

1. Verifiez que le bug n'a pas deja ete signale dans les [Issues](../../issues)
2. Creez une nouvelle issue avec:
   - Description claire du probleme
   - Etapes pour reproduire
   - Comportement attendu vs observe
   - Version de Python et des dependances

## Proposer une amelioration

1. Ouvrez une issue pour discuter de votre idee
2. Attendez la validation avant de commencer le developpement

## Soumettre du code

### Prerequis

- Python 3.10+
- [uv](https://docs.astral.sh/uv/) (gestionnaire de packages)
- [just](https://github.com/casey/just) (task runner, optionnel)

### Setup

```bash
# Cloner votre fork
git clone https://github.com/VOTRE_USERNAME/kyc-document-processing.git
cd kyc-document-processing

# Installer les dependances
uv sync

# Verifier que tout fonctionne
just check
```

### Workflow

1. Creez une branche depuis `main`:
   ```bash
   git checkout -b feature/ma-fonctionnalite
   ```

2. Faites vos modifications en suivant les conventions de code

3. Verifiez votre code:
   ```bash
   just check  # Formate, lint et tests
   ```

4. Committez avec des messages clairs:
   ```bash
   git commit -m "feat: ajouter extraction de nouveau document"
   ```

5. Poussez et creez une Pull Request

### Conventions de code

- **Formatage**: Black (ligne max 100 caracteres)
- **Linting**: Ruff
- **Types**: Annotations de type obligatoires pour les fonctions publiques
- **Tests**: Couverture minimale de 80%
- **Docstrings**: Format Google pour les fonctions publiques

### Structure des commits

Utilisez les [Conventional Commits](https://www.conventionalcommits.org/):

- `feat:` nouvelle fonctionnalite
- `fix:` correction de bug
- `docs:` documentation
- `refactor:` refactoring sans changement de comportement
- `test:` ajout ou modification de tests
- `chore:` maintenance (dependances, CI, etc.)

## Questions ?

Ouvrez une issue avec le label `question`.
