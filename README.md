# Atelier-Django

Dépôt de projet Django contenant les travaux des ateliers (Atelier 1 pushé sur `main`, Atelier 2 sur la branche `atelier-2`).
Le dépôt appartient à THEGENTLEMAN31.

> Ce README est pensé pour être édité au fur et à mesure que de nouveaux ateliers seront ajoutés — il explique la structure actuelle, comment démarrer, et donne des conseils pratiques pour la suite.

---

## Résumé / objectifs

Ce projet est une application **e-commerce pédagogique** (app `products`) utilisée pour apprendre les concepts Django : création d’un projet/app, modèles, relations (ForeignKey / ManyToMany), migrations, vues, templates, médias (images) et connexion à MySQL si besoin. Les consignes des ateliers 1 et 2 servent de base.  

---

## Branches

* `main` — contient le travail de **l'atelier 1** (actuel).
* `atelier-2` (ou `Atelier-2`) — contient **l'atelier 2**.

> Remarque : avoir des ateliers parallèles dans `main` et une branche dédiée n’est pas idéal à long terme. Idéalement chaque atelier/dev devrait être sur sa propre branche (`atelier/1`, `atelier/2`, `feature/...`) et mergé via PR quand prêt.

---

## Arborescence principale (extrait)

```
Atelier-Django/
├─ ecommerce/        # projet django (settings, urls, etc.)
├─ products/         # application products (models, views, templates)
├─ db.sqlite3        # base de données SQLite (ne devrait pas être versionnée)
├─ manage.py
└─ README.md         # (toi -> ce fichier)
```

---

## Prérequis

* Python 3.8+
* virtualenv (fortement recommandé)
* Django (version utilisée dans l'atelier — installe via `requirements.txt` si présent)
* Pillow pour la gestion d'images : `pip install pillow`. 

---

## Installation (rapide)

1. Cloner le dépôt :

```bash
git clone https://github.com/THEGENTLEMAN31/Atelier-Django.git
cd Atelier-Django
```

2. Créer et activer un environnement virtuel :

```bash
python -m venv .venv
# sous Linux/macOS
source .venv/bin/activate
# sous Windows (PowerShell)
.venv\Scripts\Activate.ps1
```

3. Installer les dépendances (si `requirements.txt` existe) :

```bash
pip install -r requirements.txt
# sinon au minimum :
pip install django pillow
```

4. Appliquer les migrations :

```bash
python manage.py makemigrations
python manage.py migrate
```

5. Créer un superuser pour l’admin :

```bash
python manage.py createsuperuser
```

6. Lancer le serveur :

```bash
python manage.py runserver
# puis ouvrir http://127.0.0.1:8000/products
```

---

## Configuration médias (images)

Le projet utilise un dossier `images/products` (ou `media/`) pour stocker les images.
Dans `settings.py` il faut définir au minimum :

```py
MEDIA_ROOT = BASE_DIR / "media"
MEDIA_URL = "/media/"
```

Et dans `ecommerce/urls.py` servir les fichiers médias en développement (voir docs de l'atelier). 

---

## Base de données

* Par défaut : **SQLite** (fichier `db.sqlite3` présent dans le repo — **ne** le versionnez pas**).
* Pour utiliser **MySQL** : configurer `DATABASES` dans `settings.py` (expliqué dans l’atelier) puis installer `mysqlclient`. 

---

## Ce que couvrent les ateliers (rapide)

* **Atelier 1** : création du projet/app, modèles (`Product`), vues, templates, configuration URLs, serveur local. 
* **Atelier 2** : gestion avancée des modèles (ajout `Category`, relations), migrations, admin, upload d’images, et passage à MySQL en option. 

---

## Bonnes pratiques & recommandations

1. **Enlever `db.sqlite3` du dépôt** et ajouter `db.sqlite3` à `.gitignore`. Garder la BD hors du repo pour éviter fuites et conflits.
2. Ajouter un `requirements.txt` (ex. `pip freeze > requirements.txt`) pour que les autres puissent installer les dépendances.
3. Créer une branche par atelier/feature : `atelier/1`, `atelier/2`, `feature/ui-login`… puis faire des pull requests pour fusionner.
4. Ne pas committer d’images volumineuses — stocker les images de démonstration dans `media/` mais ignorer les uploads utilisateur dans `.gitignore`.
5. Ajouter un fichier `.env` ou utiliser `django-environ` pour les secrets (SECRET_KEY, DB credentials). **Ne pas** committer les secrets.
6. Si le projet doit évoluer : rajouter des tests unitaires pour les modèles et vues.

---



## TODO (idées pour prochains commits)

* Supprimer `db.sqlite3` et ajouter `.gitignore`.
* Ajouter `requirements.txt`.
* Documenter chaque branche d’atelier dans le README (changelog).
* Ajouter des fixtures d’exemple (`loaddata`) pour remplir la DB avec produits & catégories.
* Mettre en place des tests unitaires pour `Product` et `Category`.

---

## Déploiement Azure App Service (Django + SQLite)

Le repo est prêt pour un déploiement **App Service Linux (Python)** via GitHub Actions.

### Fichiers liés
- `requirements.txt` : Django 6.0.3 + gunicorn + whitenoise.
- `startup.txt` : commande de démarrage Oryx (migrate → collectstatic → gunicorn).
- `.github/workflows/deploy-azure.yml` : CI/CD — chaque push sur `main` redéploie.

### À faire une fois (portail Azure)
1. **App Service** → Create → *Publish = Code*, *Runtime = Python 3.12+,* *Plan = Free (F1)* pour un test.
2. **Configuration → Startup** : Oryx lit `startup.txt` automatiquement.
3. **Configuration → Application settings** :
   | Clé | Valeur |
   |---|---|
   | `ALLOWED_HOSTS` | `<ton-app>.azurewebsites.net` |
   | `SECRET_KEY` | génère un secret : `python -c "from django.core.management.utils import get_random_secret_key; print(get_random_secret_key())"` |
   | `DEBUG` | `False` |
   | `SCM_DO_BUILD_DURING_DEPLOYMENT` | `true` (Oryx construit depuis le zip) |
   | `WEBSITES_PORT` | `8000` |
4. **Deployment Center → GitHub** (ou le workflow GA ci-dessous).

### GitHub Actions (option plus propre)
1. Télécharge le **publish profile** (App Service → Overview → *Get publish profile*).
2. GitHub → **Settings → Secrets → Actions** → nouvelle secret `AZURE_WEBAPP_PUBLISH_PROFILE` ← colle-le.
3. Renomme `AZURE_WEBAPP_NAME` dans le workflow si ton app a un autre nom.
4. Chaque push sur `main` déploie (*workflow_dispatch* pour relancer à la main).

### Base SQLite
Pense que le disque d'App Service est **éphémère** : la `db.sqlite3` est remise à zéro à chaque redéploiement. Pour du permanent par la suite : base managée (PostgreSQL/MySQL Azure) ou partage Azure Files.

---

## Licence

Par défaut : **MIT** (ou choisis celle que tu veux). Ajoute un `LICENSE` si tu souhaites partager publiquement.

---
