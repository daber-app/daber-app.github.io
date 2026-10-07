# Programmer des pop-ups dans Daber

L'app (iPhone et Android) lit le fichier `config.json` de ce site :
- au lancement ;
- à chaque retour dans l'app ;
- toutes les 30 minutes tant qu'elle reste ouverte.

Elle ne le relit pas plus d'une fois toutes les 10 minutes. Une modification publiée arrive donc chez les utilisateurs connectés en 10 à 30 minutes. Si le téléphone est hors ligne, rien ne s'affiche.

## Modifier le fichier

1. Sur github.com, ouvre le dépôt du site, puis `config.json`, puis le crayon (« Edit »).
2. Fais ta modification, puis « Commit changes ». GitHub Pages republie en une à deux minutes.
3. **Vérifie que le JSON est valide** avant de valider. Colle-le par exemple dans https://jsonlint.com.
   - Une virgule en trop ou un guillemet manquant rend tout le fichier illisible.
   - L'app n'affiche alors aucune pop-up.
   - La mise à jour forcée continue de fonctionner, car elle interroge directement l'App Store et Google Play.

## Une pop-up

```json
{
  "id": "rentree-2026",
  "title": { "fr": "Bonne rentrée !", "he": "שנה טובה ובהצלחה!" },
  "body":  { "fr": "Nouveaux textes au niveau Bet.", "he": "טקסטים חדשים ברמה ב׳." },
  "button": { "fr": "Lire", "he": "לקריאה" },
  "icon": "sparkles",
  "action": "tab:texts",
  "start": "2026-10-10",
  "end": "2026-10-20",
  "repeat": "once",
  "audience": "all",
  "levels": ["aleph", "bet"],
  "platforms": ["ios", "android"],
  "minVersion": "1.1.1"
}
```

| Champ | Obligatoire | Rôle |
|---|---|---|
| `id` | oui | Nom unique, sans espace. L'app se souvient des pop-ups déjà vues grâce à lui : **change l'`id` pour reprogrammer une pop-up déjà montrée**. |
| `title`, `body` | au moins l'un des deux | Texte en français (`fr`) et en hébreu (`he`). En interface hébraïque, l'app montre `he` s'il existe, sinon `fr`. Un texte simple (`"title": "Salut"`) compte comme du français. |
| `button` | non | Texte du bouton. « OK » par défaut. |
| `icon` | non | Nom d'icône SF Symbols, par exemple `sparkles`, `gift.fill`, `star.fill`, `bell.fill`, `book.fill`. Si le nom est inconnu, l'app affiche ✨. |
| `action` | non | Ce que fait le bouton : `premium` (écran Premium, seulement pour les non-abonnés), `tab:today`, `tab:review`, `tab:grammar`, `tab:texts`, `tab:explorer`, ou une adresse `https://…`. Sans action, le bouton ferme la pop-up. |
| `start`, `end` | non | Période d'affichage. Accepte un jour (`"2026-10-10"`) ou une heure précise (`"2026-10-10T08:00:00+02:00"`). Pour un jour seul : `start` commence à 0 h et `end` finit à 23 h 59, heure du téléphone. **Une date mal écrite fait ignorer la pop-up.** |
| `repeat` | non | `once` : une seule fois (par défaut). `daily` : une fois par jour. `weekly` : une fois par semaine. `always` : à chaque ouverture. |
| `audience` | non | `all` (par défaut), `free` (non-abonnés seulement) ou `premium` (abonnés seulement). |
| `levels` | non | Limite la pop-up à certains niveaux : `alphabet`, `aleph`, `bet`, `gimel`, `dalet`. |
| `platforms` | non | `ios`, `android`, ou les deux. Par défaut, les deux. |
| `minVersion`, `maxVersion` | non | Limite la pop-up à certaines versions de l'app. Par exemple `"maxVersion": "1.1.1"` pour ne parler qu'aux anciennes versions. |

Règles d'affichage :
- L'app montre **au plus une pop-up par ouverture** : la première de la liste qui convient. L'ordre du fichier donne donc la priorité.
- Une pop-up n'apparaît jamais pendant l'onboarding, pendant le tutoriel, par-dessus un autre écran ouvert ou par-dessus l'écran de mise à jour.
- Quand une pop-up s'affiche, l'invitation automatique à Premium attend le lendemain.

Plusieurs pop-ups se mettent les unes après les autres dans `messages`, séparées par des virgules :

```json
"messages": [
  { "id": "promo-hanoucca", "title": { "fr": "…" }, "start": "2026-12-04", "end": "2026-12-12", "audience": "free", "action": "premium" },
  { "id": "nouveaux-dialogues", "title": { "fr": "…" }, "repeat": "once", "action": "tab:texts" }
]
```

## Mise à jour forcée

Par défaut, dès que l'App Store (ou Google Play) propose une version plus récente que celle installée, l'app affiche « Faire la mise à jour ». Cet écran ne se ferme pas tant que la mise à jour n'est pas faite.

Le bloc `update` sert de télécommande :
- `"force": false` **désactive le blocage** automatique. C'est l'issue de secours si Apple annonce une version avant qu'elle soit téléchargeable partout, ou en cas de problème. Remets `true` ensuite.
- `"minimumVersion": { "ios": "1.2", "android": "1.2" }` bloque toutes les versions inférieures, même si `force` vaut `false`. Mets `null` pour ne rien exiger.

Apple met jusqu'à 24 h à mettre à jour l'information de version dans tous les pays. Le blocage peut donc commencer quelques heures après la mise en ligne.
