# Outils de développement et contribution

Ce document décrit les outils et standards utilisés pour le développement de ce projet, ainsi que les étapes pour configurer votre environnement de travail et contribuer efficacement.

## Outils et standard d'équipe

- **GitHub Actions** (`.github/workflows/unit-tests.yml`): exécute automatiquement les tests à chaque push/PR pour garantir que tout le monde valide le même niveau de qualité avant fusion.
- **pytest** (groupe `dev` dans `pyproject.toml`): framework de tests unitaires, utilisé pour vérifier le comportement du code de manière reproductible entre les membres.
- **pre-commit** (`.pre-commit-config.yaml`): au moment de commit, l'outil `pre-commit` exécute les hooks configurés, pour garantir que le code respecte les standards de qualité avant même d'être poussé.
- **Ruff** (`pre-commit` + groupe `dev` dans `pyproject.toml`): linter et formateur de code Python qui remplace le duo historique Black/isort, pour garder un code lisible et uniforme.
- **Séparation des dépendances** (`pyproject.toml` + `uv.lock`): distingue les dépendances nécessaires pour exécuter le projet de celles réservées au développement, tout en gardant un verrouillage reproductible des versions.

## Première configuration

> [!IMPORTANT]
> Ce template est prévu pour `uv`. Utilisez `uv sync` pour installer les dépendances et `uv run` pour exécuter les commandes du projet.

### Configuration avec [`uv`](https://docs.astral.sh/uv/)

<details>
<summary>Instructions pour uv</summary>

#### 1) Installer les dépendances

```bash
uv sync --group dev
```

#### 2) Exécuter les tests

```bash
uv run pytest tests/
```

#### 3) Lancer les hooks qualité

```bash
uv tool install pre-commit
uvx pre-commit install
uvx pre-commit run --all-files
```

#### 4) Lancer le script d'exemple

```bash
uv run python src/main.py
```

> [!TIP]
> Si vous préférez ne pas installer `pre-commit` globalement, vous pouvez remplacer `uvx pre-commit ...` par une commande équivalente dans un environnement où `pre-commit` est déjà installé.

</details>

## Contribution

- **Développement**: travaillez dans `src/` pour le code d'application, et dans `tests/` pour les tests unitaires.
- **Tests**: lancez les tests avec pytest via `uv run pytest tests/` pour vérifier que votre code fonctionne comme prévu.
- **Qualité**: les hooks `pre-commit` s'exécutent automatiquement au moment du commit, avec Ruff comme formateur et linter.
- **CI**: à chaque push sur `main` ou `dev`, et à chaque Pull Request, les tests sont exécutés automatiquement sur GitHub Actions pour garantir que le code fusionné passe les tests.
- **Documentation**: ajoutez des commentaires et des docstrings (Google Style) pour expliquer le fonctionnement de votre code, surtout si vous travaillez en équipe.
- **Type-hints**: utilisez des annotations de type pour faciliter la lecture par les pairs et l'IA si vous en utilisez.

## Ressources supplémentaires

### Bibliothèques utilisées

- [`deeplenstronomy`]()

### Outils de développement

- [Documentation pytest](https://docs.pytest.org/en/stable/)
- [Documentation pre-commit](https://pre-commit.com/)
- [Documentation Ruff](https://docs.astral.sh/ruff/)
- [Documentation pre-commit Ruff hooks](https://github.com/astral-sh/ruff-pre-commit)
- [PEP 8 - Style Guide for Python Code](https://www.python.org/dev/peps/pep-0008/)

### Gestion d'environnement

- [uv - Documentation](https://docs.astral.sh/uv/)