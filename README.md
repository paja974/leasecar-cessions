# Outil de Cessions Véhicules — LEASECAR

Outil web single-file pour la génération automatique des 3 Cerfa de cession véhicule
à partir de la carte grise. Développé par 10Positif pour LEASECAR.

## Contenu du dossier

- `cession.html` — l'application complète (à ouvrir dans un navigateur)
- `cerfa_13751.pdf` — Cerfa officiel "Déclaration d'achat" (banque → LEASECAR)
- `cerfa_15776.pdf` — Cerfa officiel "Certificat de cession" (LEASECAR → client)
- `cerfa_13754.pdf` — Cerfa officiel "Déclaration de cession" (archive)

⚠️ **Garder les 4 fichiers dans le même dossier**. Le HTML va chercher les 3 PDF
à côté de lui pour les remplir.

## Workflow opérationnel

Une cession LEASECAR se déroule en 2 temps administratifs :

1. **Banque → LEASECAR** : la banque qui détenait le véhicule (BNP Lease, Arval,
   CGI, Crédit Agricole Leasing, etc.) cède le véhicule à LEASECAR.
   → **Cerfa 13751** (déclaration d'achat par un professionnel)

2. **LEASECAR → Client final** : LEASECAR revend en son nom propre au client.
   → **Cerfa 15776** (certificat de cession véhicule d'occasion)
   → **Cerfa 13754** (ancienne version, conservée en archive)

L'outil génère les 3 PDF en une seule opération, qu'il suffit ensuite d'imprimer
et tamponner.

## Déploiement

### Option A — Usage local (le plus simple)
1. Décompresser le dossier sur le poste du collaborateur (ex. `Documents/cession/`)
2. Double-cliquer sur `cession.html` → ouverture dans Chrome/Edge/Firefox
3. C'est tout. 100% offline, aucune donnée transmise.

### Option B — Hébergement partagé (intranet / GitHub Pages)
1. Pousser les 4 fichiers dans un repo GitHub
2. Activer GitHub Pages (Settings → Pages → Source: main branch)
3. Tous les collaborateurs accèdent via une URL `https://[org].github.io/cession/`

### Option C — Serveur interne 10Positif
Servir le dossier statique via Nginx/Apache. Aucun backend requis.

## Première utilisation — Configuration

Ouvrir `cession.html` dans un éditeur de texte et éditer les sections en haut
du script (ligne ~480 environ) :

### 1. Coordonnées LEASECAR
```js
const LEASECAR_DEFAULT = {
  nom: 'LEASECAR',
  siren: '',                    // ← compléter avec le SIREN officiel
  voie: '103',
  typevoie: 'AVENUE',
  nomvoie: 'JACQUES PREVERT',
  cp: '97420',
  commune: 'LE PORT'
};
```

### 2. Liste des banques partenaires
```js
const BANQUES = [
  {
    id: 'bnp_lease',
    label: 'BNP Paribas Lease Group',   // libellé visible dans le menu
    nom: 'BNP PARIBAS LEASE GROUP',      // raison sociale (sur le Cerfa)
    siren: '632017513',
    voie: '46',
    typevoie: 'RUE',
    nomvoie: 'DE PROVENCE',
    cp: '75009',
    commune: 'PARIS'
  },
  // ajouter autant de banques que nécessaire
];
```

Les SIREN fournis par défaut dans le code sont indicatifs — **vérifier et
remplacer** par les vraies coordonnées des contrats LEASECAR.

## Utilisation quotidienne

1. **Scanner / photographier la carte grise** (recto bien net, contraste correct)
2. **Déposer l'image** dans la zone de drop ou cliquer pour l'ouvrir
3. L'OCR lit la carte grise → les champs Immatriculation, VIN, marque, modèle,
   etc. se remplissent automatiquement (fond vert clair)
4. **Vérifier et corriger** les champs si nécessaire (l'OCR n'est pas parfait)
5. **Sélectionner la banque cédante** dans le menu déroulant → ses coordonnées
   se pré-remplissent
6. **Saisir les coordonnées de l'acquéreur final** (client)
7. Décocher éventuellement un des 3 documents si non nécessaire
8. **Cliquer "Générer les PDF"**

### Sauvegarde des PDF générés

Deux options s'offrent à vous :

#### 📁 Option recommandée : "Enregistrer dans un dossier…"
Un clic ouvre une boîte de dialogue pour choisir le dossier de destination
(par exemple `Cessions/2026/`). L'outil crée alors automatiquement un
sous-dossier au format `2026-05-27_AB-123-CD_DUPONT/` contenant les 3 PDF
nommés proprement. Idéal pour archiver par dossier client.

> ⚠️ **Cette fonctionnalité nécessite Chrome ou Edge** sur desktop, ET un
> contexte sécurisé (https:// ou localhost). Elle ne fonctionne PAS via
> `file://` (double-clic sur l'HTML depuis l'explorateur Windows).
>
> Pour l'usage local, voir la section "Lancer en local sans serveur" plus bas.

#### ⬇ Option fallback : liens de téléchargement individuels
Toujours disponibles, les 3 liens téléchargent les PDF un par un dans le
dossier "Téléchargements" du navigateur (avec leur nom auto :
`15776_cession_AB-123-CD.pdf`, etc.). Compatible tous navigateurs.

9. **Imprimer, tamponner, faire signer**

## Lancer en local sans serveur (Windows / Mac)

Pour avoir le bouton "Enregistrer dans un dossier…" en usage local :

### Windows
1. Ouvrir une invite de commandes (`cmd`) dans le dossier de l'outil
2. Lancer : `python -m http.server 8080`
   (si Python n'est pas installé : `winget install Python.Python.3`)
3. Ouvrir http://localhost:8080/cession.html dans Chrome ou Edge

### Mac / Linux
1. Terminal dans le dossier de l'outil
2. `python3 -m http.server 8080`
3. Ouvrir http://localhost:8080/cession.html

### Solution la plus simple : créer un raccourci
Créer un fichier `lancer.bat` (Windows) à côté de `cession.html` :

```bat
@echo off
cd /d "%~dp0"
start http://localhost:8080/cession.html
python -m http.server 8080
```

Double-clic sur `lancer.bat` → le navigateur s'ouvre, le serveur tourne,
tout fonctionne. Fermer la fenêtre `cmd` pour arrêter.

## Maintenance / Évolutions

### Ajuster les positions de texte sur les Cerfa
Si un nouveau Cerfa officiel est publié et que le texte se retrouve mal aligné,
ouvrir `cession.html` et chercher les fonctions `fillCerfa15776`, `fillCerfa13754`
et `fillCerfa13751`. Les coordonnées sont en points PDF (système y-from-top,
A4 = 595×842 pt).

### Ajouter un nouveau type de document
Dupliquer une fonction `fillCerfa*` existante, l'adapter, et ajouter une case
à cocher dans la section "Documents à générer" du HTML.

### Limitations connues
- **OCR** : Tesseract.js fonctionne bien sur des scans nets, moins bien sur des
  photos prises au téléphone (mauvais cadrage, reflets). Les champs sont toujours
  éditables manuellement.
- **Format de l'immatriculation** : seul le format SIV récent (`AB-123-CD`) est
  parfaitement reconnu. L'ancien format (`123 AB 974`) demande une saisie manuelle.
- **Le navigateur doit autoriser fetch() sur le file://** pour usage local. En cas
  de blocage, héberger les fichiers via un serveur local (`python -m http.server`).

## Confidentialité

Aucune donnée n'est transmise à un serveur. Tout (OCR, parsing, génération PDF)
se passe dans le navigateur du collaborateur. Conforme RGPD par construction.

---

*v2.0 · 10Positif SARL · La Réunion (974) · 2026*
