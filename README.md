# DevArena

# DevArena — Club Dev ESGIS Bénin

DevArena est une plateforme de défis de programmation chronométrés avec classement en temps réel. Conçue pour la journée de lancement des clubs de l'ESGIS Bénin, elle permet aux étudiants de tester leurs compétences en code sur stand pendant que les scores s'affichent en direct sur vidéoprojecteur.

## Fonctionnalités principales

- **Arène Joueur (`/play`)** : Interface scindée avec consigne et chronomètre, éditeur de code intégré (coloration syntaxique) et prévisualisation en direct (`iframe` isolée).
- **5 Niveaux progressifs** : Un parcours rapide (6 min 30 max) allant de l'affichage HTML de base jusqu'à l'algorithmique.
- **Moteur de validation automatique** : Vérification instantanée du DOM, du style CSS et de la logique JavaScript du joueur.
- **Projecteur Temps Réel (`/leaderboard`)** : Tableau des scores optimisé pour grand écran avec podium Top 3, classement par filière/promo et mise à jour instantanée via WebSockets.

## Structure des 5 niveaux et scoring

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
- **Base de données & Realtime** : Supabase (PostgreSQL + Realtime Subscriptions)
- **Hébergement** : Vercel

## Architecture du projet

```text
devarena/
├── src/
│   ├── app/
│   │   ├── page.tsx              # Accueil et inscription (Pseudo + Filière)
│   │   ├── play/page.tsx         # Arène de code (Éditeur + Preview + Chrono)
│   │   └── leaderboard/page.tsx  # Écran vidéoprojecteur temps réel
│   ├── components/
│   │   ├── CodeEditor.tsx        # Wrapper CodeMirror
│   │   ├── LivePreview.tsx       # Sandbox iframe + exécution des tests
│   │   └── ScoreBoardTable.tsx   # Tableau animé du classement
│   ├── lib/
│   │   ├── levels.ts             # Définitions, codes initiaux et assertions des 5 niveaux
│   │   └── supabase.ts           # Client Supabase
└── .env.local                    # Variables d'environnement
```

## Installation en local

### 1. Cloner le dépôt et installer les dépendances

```bash
git clone [https://github.com/](https://github.com/)<votre-org>/devarena.git
cd devarena
npm install
```

### 2. Configurer les variables d'environnement

Crée un fichier `.env.local` à la racine du projet et renseigne tes clés Supabase :

```env
NEXT_PUBLIC_SUPABASE_URL=[https://votre-projet.supabase.co](https://votre-projet.supabase.co)
NEXT_PUBLIC_SUPABASE_ANON_KEY=votre-cle-publique-anon
```

### 3. Configuration de la base de données (Supabase)

Exécute cette requête SQL dans l'éditeur SQL de ton projet Supabase et active Realtime sur la table `scores` :

```sql
CREATE TABLE scores (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  pseudo TEXT NOT NULL,
  filiere TEXT NOT NULL,
  score INTEGER DEFAULT 0,
  current_level INTEGER DEFAULT 1,
  total_time_seconds INTEGER DEFAULT 0,
  updated_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Activer la lecture/écriture publique pour l'événement
ALTER TABLE scores ENABLE ROW LEVEL SECURITY;
CREATE POLICY "Public access" ON scores FOR ALL USING (true) WITH CHECK (true);
```

### 4. Lancer le serveur de développement

```bash
npm run dev
```

Ouvre `http://localhost:3000/play` pour tester le jeu et `http://localhost:3000/leaderboard` dans un second onglet pour vérifier la synchronisation en direct.

## Workflow Git pour l'équipe

1. Ne jamais pousser directement sur `main` (branche connectée à la production Vercel).
2. Créer une branche par fonctionnalité :
   - `feat/player-ui`
   - `feat/validation-engine`
   - `feat/leaderboard-screen`
3. Ouvrir une Pull Request vers `main` dès qu'un module est fonctionnel et testé.

---

Club Dev — ESGIS Bénin
