
# Gestion des notes d'une faculté (FSO)

Plateforme web de gestion des notes : saisie des notes par les professeurs, calcul automatique des moyennes, publication et consultation des résultats par les étudiants, administration des filières, modules et comptes.

## Équipe

| Membre | Rôle dans le projet |
|---|---|
| Dounia Zirari | À compléter |
| [Prénom Nom de votre collègue] | À compléter |

Encadrant : [Nom du professeur]

## Technologies

- **Python** : logique de l'application
- **MySQL** : base de données
- **HTML** et **CSS** : interface
- **Git / GitHub** : suivi du travail

## Fonctionnalités

- **Étudiant** : consulter ses notes par semestre, sa moyenne et son résultat (V / RAT / NV), télécharger son relevé de notes.
- **Professeur** : consulter la liste de ses étudiants par module, saisir les notes (CC, TP, examen), valider et publier les notes. Les moyennes sont calculées automatiquement.
- **Administrateur** : gérer les comptes, les filières et les modules, affecter les professeurs aux modules, superviser et verrouiller les délibérations.
- **Tous les utilisateurs** doivent s'authentifier avant d'accéder à une fonctionnalité.

## Diagramme de cas d'utilisation

Les trois acteurs principaux (étudiant, professeur, administrateur) sont à gauche. Chaque fonctionnalité inclut le cas « S'authentifier ». Le calcul des moyennes est inclus dans la validation et la publication des notes.

```mermaid
flowchart LR
    E["Étudiant"]
    P["Professeur"]
    A["Administrateur"]

    subgraph SYS["Plateforme de gestion des notes (FSO)"]
        E1(["Consulter ses notes par semestre"])
        E2(["Télécharger son relevé de notes"])
        E3(["Consulter sa moyenne et résultat"])

        P1(["Consulter ses étudiants par module"])
        P2(["Saisir les notes (CC, TP, examen)"])
        P3(["Valider et publier les notes"])

        A1(["Gérer les comptes utilisateurs"])
        A2(["Gérer les filières et modules"])
        A3(["Affecter professeurs aux modules"])
        A4(["Superviser les délibérations"])

        P4(["Calculer les moyennes"])
        AUTH(["S'authentifier"])
    end

    E --- E1
    E --- E2
    E --- E3
    P --- P1
    P --- P2
    P --- P3
    A --- A1
    A --- A2
    A --- A3
    A --- A4

    P3 -.->|"«include»"| P4

    E1 -.->|"«include»"| AUTH
    E2 -.->|"«include»"| AUTH
    E3 -.->|"«include»"| AUTH
    P1 -.->|"«include»"| AUTH
    P2 -.->|"«include»"| AUTH
    P3 -.->|"«include»"| AUTH
    A1 -.->|"«include»"| AUTH
    A2 -.->|"«include»"| AUTH
    A3 -.->|"«include»"| AUTH
    A4 -.->|"«include»"| AUTH
```

Version dessinée du même diagramme, avec en plus la relation « extend » entre le téléchargement du relevé et la consultation des notes :

![Diagramme de cas d'utilisation](docs/use-case.svg)

## Diagramme de classes

```mermaid
classDiagram
    class Utilisateur {
        -int id
        -String nom
        -String prenom
        -String email
        -String login
        -String motDePasseHash
        -String role
        +verifierMotDePasse() boolean
    }
    class Administrateur {
        -String grade
        +creerUtilisateur()
        +gererFilieresModules()
        +affecterProfesseur()
        +verrouillerDeliberations()
    }
    class Professeur {
        -String matricule
        -String departement
        -String telephone
        +consulterEtudiants()
        +saisirNotes()
        +validerNotes()
    }
    class Etudiant {
        -String codeApogee
        -String cne
        -String cin
        -Date dateNaissance
        +consulterNotes()
        +telechargerReleve()
    }
    class Filiere {
        -int id
        -String codeFiliere
        -String intitule
        -String departement
    }
    class Semestre {
        -int id
        -String codeSemestre
        -String libelle
    }
    class Module {
        -int id
        -String codeModule
        -String intitule
        -float coefCc
        -float coefTp
        -float coefExam
    }
    class Affectation {
        -int id
        -String anneeUniversitaire
    }
    class Note {
        -int id
        -String anneeUniversitaire
        -float noteCc
        -float noteTp
        -float noteExam
        -float noteRattrapage
        -float moyenneFinale
        -String statut
        -boolean estValidee
        -boolean estPubliee
        -boolean verrouille
        +calculerMoyenne()
        +determinerStatut()
    }

    Utilisateur <|-- Administrateur
    Utilisateur <|-- Professeur
    Utilisateur <|-- Etudiant

    Filiere "1" --> "0..*" Etudiant : regroupe
    Filiere "1" --> "1..*" Module : comporte
    Semestre "1" --> "1..*" Module : organise
    Professeur "1" --> "0..*" Affectation : enseigne via
    Module "1" --> "0..*" Affectation : est dispensé via
    Etudiant "1" --> "0..*" Note : obtient
    Module "1" --> "0..*" Note : concerne
```

Remarque : les diagrammes Mermaid sont dessinés par GitHub à partir de leur code texte. Ils servent uniquement à la documentation et ne font pas partie de l'application.

## Structure du projet

```
gestion-notes-faculte/
├── app.py                 # programme principal en Python
├── database/
│   └── schema.sql         # création de la base MySQL
├── templates/             # pages HTML
├── static/
│   └── css/style.css      # styles CSS
├── .gitignore
└── README.md
```

## Installation

1. Cloner le dépôt :
   ```bash
   git clone <lien-HTTPS-du-depot>
   cd gestion-notes-faculte
   ```
2. Créer la base de données (MySQL 8.0.16 ou plus récent) :
   ```bash
   mysql -u root -p < database/schema.sql
   ```
3. Installer le connecteur MySQL pour Python :
   ```bash
   pip install mysql-connector-python
   ```
4. Copier `config.example.py` en `config.py` et y renseigner les paramètres de connexion MySQL.
   Le fichier `config.py` ne doit **jamais** être envoyé sur GitHub.
5. Lancer l'application :
   ```bash
   python app.py
   ```

## Règle de calcul de la moyenne

```
moyenne = note_cc × coef_cc + note_tp × coef_tp + note_exam × coef_exam
```

- `V` (validé) : moyenne supérieure ou égale à 10
- `RAT` (rattrapage) : moyenne inférieure à 10, rattrapage possible
- `NV` (non validé) : moyenne toujours inférieure à 10 après le rattrapage

## Répartition des tâches

| Tâche | Responsable | État |
|---|---|---|
| Diagrammes UML | À compléter | Terminé |
| Base de données MySQL | À compléter | En cours |
| Authentification | À compléter | À faire |
| Interface professeur | À compléter | À faire |
| Interface étudiant | À compléter | À faire |
| Interface administrateur | À compléter | À faire |
