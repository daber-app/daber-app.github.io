# Les pop-ups de Daber

Tu les pilotes depuis un seul fichier : `config.json`, sur ce site. L'app iPhone et l'app Android le lisent :
- au lancement ;
- à chaque retour dans l'app ;
- toutes les 30 minutes tant qu'elle reste ouverte.

Une modification arrive en 10 à 30 minutes chez les utilisateurs connectés. Hors ligne, rien ne s'affiche.

Il y a **deux types de pop-up** :

| Type | Pour qui | Combien de fois |
|---|---|---|
| **Mise à jour** (`"type": "update"`) | Seulement les personnes qui viennent de **mettre à jour** l'app vers la version indiquée. Quelqu'un qui installe l'app pour la première fois ne la voit pas. | **Une seule fois**. Une fois fermée, plus jamais. |
| **Message** (par défaut) | Tous les utilisateurs, ou ceux que tu choisis (gratuits, abonnés, niveaux, iPhone ou Android, versions). | Ce que tu décides : une fois, chaque jour, chaque semaine ou à chaque ouverture, entre deux dates si tu veux. |

## Modifier le fichier

1. Sur github.com, ouvre le dépôt `daber-app/daber-app.github.io`, puis le fichier `config.json`, puis le crayon (« Edit »).
2. Fais ta modification, puis « Commit changes ». Le site se met à jour en une à deux minutes.
3. **Vérifie que le fichier est valide** en le collant sur https://jsonlint.com.
   - Une virgule en trop ou un guillemet manquant rend tout le fichier illisible, et alors aucune pop-up ne s'affiche.
   - La mise à jour forcée, elle, continue de marcher.

Les pop-ups se placent dans la liste `"messages": [ … ]`, séparées par des virgules. L'app montre **au plus une pop-up par ouverture** : la première de la liste qui convient. L'ordre du fichier donne donc la priorité.

## Type 1 — pop-up de mise à jour

À mettre dans le fichier **avant** la sortie de la version. Elle reste invisible jusqu'à ce que les gens installent cette version.

```json
{
  "id": "bienvenue-1.1.1",
  "type": "update",
  "version": "1.1.1",
  "title": { "fr": "Bienvenue dans la mise à jour 1.1.1 !", "he": "ברוכים הבאים לעדכון 1.1.1!" },
  "body": { "fr": "Quoi de neuf ?\n\n• Première nouveauté\n• Deuxième nouveauté", "he": "מה חדש?\n\n• …\n• …" },
  "button": { "fr": "C'est parti !", "he": "יאללה!" },
  "icon": "sparkles"
}
```

- `version` est obligatoire. La pop-up ne s'affiche que sur cette version exacte : une personne qui passe directement de la 1.1 à la 1.2 ne verra pas celle de la 1.1.1.
- Pas besoin de `repeat`, de dates ni de version min/max : c'est toujours une fois, et seulement pour ceux qui ont mis à jour.
- À la version suivante, ajoute une nouvelle pop-up `bienvenue-1.2` avec `"version": "1.2"`. Tu peux supprimer l'ancienne ou la laisser, elle ne gêne pas.

## Type 2 — message programmé

```json
{
  "id": "promo-hanoucca-2026",
  "title": { "fr": "Joyeux Hanoucca !", "he": "חג חנוכה שמח!" },
  "body": { "fr": "Profite de l'hiver pour avancer.", "he": "…" },
  "button": { "fr": "Découvrir Premium", "he": "לגלות את פרימיום" },
  "icon": "gift.fill",
  "action": "premium",
  "start": "2026-12-04",
  "end": "2026-12-12",
  "repeat": "daily",
  "audience": "free"
}
```

| Champ | Obligatoire | Rôle |
|---|---|---|
| `id` | oui | Nom unique, sans espace. L'app se souvient de ce qu'elle a déjà montré grâce à lui. **Change l'`id`** pour remontrer une pop-up déjà vue. |
| `title`, `body` | au moins l'un des deux | Texte en français (`fr`) et en hébreu (`he`). Dans le texte, `\n` passe à la ligne, et une ligne qui commence par `• ` s'affiche en liste. |
| `button` | non | Texte du bouton. « OK » par défaut. |
| `icon` | non | Icône : `sparkles`, `gift.fill`, `star.fill`, `bell.fill`, `book.fill`, `heart.fill`, `megaphone.fill`, `flame.fill`, `crown.fill`, `calendar`. ✨ par défaut. |
| `action` | non | Ce que fait le bouton : `premium` (écran Premium, seulement pour les non-abonnés), `placement` (ouvre le test de niveau), `tab:today`, `tab:review`, `tab:grammar`, `tab:texts`, `tab:explorer`, ou un lien `https://…`. Sans action, le bouton ferme la pop-up. |
| `start`, `end` | non | Période d'affichage. Accepte un jour (`"2026-12-04"`) ou une heure précise (`"2026-12-04T18:00:00+02:00"`). Pour un jour seul, `start` commence à 0 h et `end` finit à 23 h 59, heure du téléphone. **Une date mal écrite fait ignorer la pop-up.** |
| `repeat` | non | `once` : une seule fois (par défaut). `daily` : une fois par jour. `weekly` : une fois par semaine. `always` : à chaque ouverture. |
| `audience` | non | `all` (par défaut), `free` (non-abonnés seulement) ou `premium` (abonnés seulement). |
| `levels` | non | Seulement certains niveaux : `alphabet`, `aleph`, `bet`, `gimel`, `dalet`. |
| `platforms` | non | `ios` ou `android`. Par défaut, les deux. |
| `minVersion`, `maxVersion` | non | Seulement certaines versions de l'app. |

Les pop-ups n'apparaissent jamais pendant l'onboarding ou le tutoriel, ni par-dessus un écran ouvert ou l'écran de mise à jour. Quand une pop-up s'affiche, l'invitation automatique à Premium attend le lendemain.

## Mise à jour forcée

Dès que l'App Store (ou Google Play) propose une version plus récente que celle installée, l'app affiche « Faire la mise à jour ». Cet écran ne se ferme pas tant que la mise à jour n'est pas faite.

Le bloc `update` du fichier sert de télécommande :
- `"force": false` **désactive le blocage**. C'est l'issue de secours si Apple annonce une version avant qu'elle soit téléchargeable partout. Remets `true` ensuite.
- `"minimumVersion": { "ios": "1.2", "android": "1.2" }` bloque les versions inférieures, même si `force` vaut `false`. Mets `null` pour ne rien exiger.

Apple met jusqu'à 24 h à diffuser la nouvelle version partout. Le blocage peut donc démarrer quelques heures après la mise en ligne.
