# DevArena — Club Dev ESGIS Bénin

DevArena est une plateforme open-source de défis de programmation chronométrés avec classement en temps réel. Conçue pour la journée de lancement des clubs de l'ESGIS Bénin, elle permet aux étudiants de tester leurs compétences sur stand pendant que les scores s'affichent en direct sur vidéoprojecteur.

## Fonctionnalités principales

- **Arène Joueur (`/play`)** : Interface scindée avec consigne et chronomètre, éditeur de code intégré (coloration syntaxique) et prévisualisation en direct (`iframe` isolée).
- **5 Niveaux progressifs** : Un parcours rapide (6 min 30 max) allant du HTML de base jusqu'à l'algorithmique.
- **Validation côté serveur** : Vérification des réponses via Next.js Server Actions / API Routes pour empêcher la lecture des solutions dans le code client.
- **Projecteur Temps Réel (`/leaderboard`)** : Tableau des scores optimisé pour grand écran avec podium Top 3, classement par filière et mise à jour instantanée.

## Structure des niveaux et scoring

| Niveau | Thème | Chrono | Points de base | Objectif |
| :--- | :--- | :---: | :---: | :--- |
| 1. Le Déclic | HTML | 45s | 100 pts | Afficher un titre exact (`Hello ESGIS`). |
| 2. Le Styliste | CSS | 60s | 150 pts | Styliser un bouton aux couleurs de l'ESGIS. |
| 3. L'Interactif | JS (DOM) | 90s | 200 pts | Implémenter un bouton compteur (`+1` au clic). |
| 4. Bug Hunter | Debug | 90s | 250 pts | Corriger 2 erreurs cachées dans un script existant. |
| 5. Le Boss Final | Algo | 120s | 300 pts | Résoudre un mini-problème de logique pure. |

### Règles de calcul des points

- **Bonus de rapidité** : +2 pts par seconde restante au chronomètre lors de la validation.
- **Bouton Indice (Hint)** : -30 pts sur le niveau en cours.
- **Bouton Passer (Skip)** : Disponible après 30s (0 pt attribué pour le niveau).
- **Départage** : En cas d'égalité de score, le joueur ayant réalisé le temps total le plus court est classé devant.

## Stack technique

- **Framework** : Next.js (App Router) & TypeScript
- **Styling** : Tailwind CSS
- **Éditeur de code** : `@uiw/react-codemirror`
- **Base de données & Realtime** : Supabase (PostgreSQL + Realtime)
- **Hébergement** : Vercel

## Architecture du projet

```text
devarena/
├── src/
│   ├── app/
│   │   ├── api/validate/route.ts # Validation sécurisée des niveaux et mise à jour des scores
│   │   ├── page.tsx              # Accueil et inscription (Pseudo + Filière)
│   │   ├── play/page.tsx         # Arène de code (Éditeur + Preview + Chrono)
│   │   └── leaderboard/page.tsx  # Écran vidéoprojecteur temps réel
│   ├── components/
│   │   ├── CodeEditor.tsx        # Wrapper CodeMirror
│   │   ├── LivePreview.tsx       # Sandbox iframe
│   │   └── ScoreBoardTable.tsx   # Tableau du classement
│   ├── lib/
│   │   ├── levels.ts             # Énoncés et codes de départ publics
│   │   └── supabase.ts           # Clients Supabase (public & admin serveur)
├── .env.example                  # Modèle des variables d'environnement
└── .gitignore                    # Exclusion des fichiers sensibles (.env.local)
```

> **Moteur de validation (pôle 2)** : les règles de correction, le calcul des
> points et les tests cachés sont documentés dans
> [`docs/validation-engine.md`](docs/validation-engine.md). Lancer `npm test`
> pour vérifier les 5 niveaux et le barème.

## Installation en local

### 1. Cloner le dépôt et installer les dépendances

```bash
git clone [https://github.com/](https://github.com/)<votre-org>/devarena.git
cd devarena
npm install
```

### 2. Configurer les variables d'environnement

Copie le fichier `.env.example` vers `.env.local` et remplis tes clés Supabase (ne jamais commiter `.env.local`) :

```bash
cp .env.example .env.local
```

```env
NEXT_PUBLIC_SUPABASE_URL=[https://votre-projet.supabase.co](https://votre-projet.supabase.co)
NEXT_PUBLIC_SUPABASE_ANON_KEY=votre-cle-publique-anon
SUPABASE_SERVICE_ROLE_KEY=votre-cle-secrete-serveur
```

### 3. Configuration de la base de données (Supabase)

Puisque ce dépôt est public, l'écriture directe dans la table `scores` depuis le navigateur est bloquée. Seule l'API serveur Next.js (via `SUPABASE_SERVICE_ROLE_KEY`) peut insérer ou modifier un score.

Exécute cette requête dans l'éditeur SQL de Supabase et active Realtime sur la table `scores` :

```sql
CREATE TABLE scores (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  pseudo TEXT NOT NULL CHECK (char_length(pseudo) BETWEEN 2 AND 20),
  filiere TEXT NOT NULL,
  score INTEGER DEFAULT 0 CHECK (score >= 0 AND score <= 2000),
  current_level INTEGER DEFAULT 1,
  total_time_seconds INTEGER DEFAULT 0,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

ALTER TABLE scores ENABLE ROW LEVEL SECURITY;

-- Lecture publique autorisée pour le leaderboard, écriture bloquée côté client
CREATE POLICY "Public read access" ON scores FOR SELECT USING (true);
```

### 4. Lancer le serveur de développement

```bash
npm run dev
```

Ouvre `http://localhost:3000/play` pour tester l'arène et `http://localhost:3000/leaderboard` dans un second onglet pour vérifier la synchronisation en direct.

## Contribuer au projet

1. Forke le dépôt ou crée une branche dédiée depuis `main` :
   - `feat/player-ui`
   - `feat/validation-engine`
   - `feat/leaderboard-screen`
2. Assure-toi qu'aucune clé API ou fichier `.env` n'est inclus dans tes commits.
3. Ouvre une Pull Request claire décrivant les modifications apportées.

## Licence

Projet distribué sous licence MIT — Club Dev ESGIS Bénin.
