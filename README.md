# Mercato OM 2026 — Outil drag & drop « Je garde / Je vends »

Outil interactif (1 seul fichier `index.html`, sans dépendance) qui permet aux supporters
de choisir, par glisser-déposer, quels joueurs de l'effectif **Olympique de Marseille 2025/2026**
ils gardent ou vendent pour la saison 2026/2027, puis de **partager** leur sélection.

- Effectif récupéré via l'**API Sportmonks** (photos, noms, postes) et figé dans le fichier
  (la clé API **n'est pas** exposée dans le HTML livré).
- Valeurs marchandes **estimées** (indicatives) → la colonne « Je vends » cumule le montant récupérable.
- Drag & drop **souris + tactile** (mobile), boutons rapides « Garder / Vendre » en secours.
- Écran de résultat + partage **X, Facebook, WhatsApp, Web Share (mobile), copier le texte**.

## 1. Héberger sur GitHub Pages

1. Crée un dépôt GitHub et pousse `index.html` à la racine.
2. `Settings` → `Pages` → Source : `Deploy from a branch`, branche `main`, dossier `/ (root)`.
3. L'outil sera dispo sur `https://<utilisateur>.github.io/<repo>/`.

## 2. Intégrer dans WordPress (bloc HTML personnalisé)

```html
<iframe
  id="mercato-om"
  src="https://<utilisateur>.github.io/<repo>/"
  style="width:100%;border:0;min-height:1400px;"
  loading="lazy"
  title="Mercato OM 2026"></iframe>

<!-- Optionnel : ajuste automatiquement la hauteur de l'iframe -->
<script>
window.addEventListener('message', function (e) {
  if (e.data && e.data.type === 'mercato-om-height') {
    document.getElementById('mercato-om').style.height = e.data.height + 'px';
  }
});
</script>
```

> L'outil envoie sa hauteur au parent via `postMessage` ; le script ci-dessus la répercute
> sur l'iframe. Sans ce script, garde simplement un `min-height` suffisant.

## 3. Personnaliser

Tout est en haut de la balise `<script>` dans `index.html` :

- **`CONFIG.shareUrl`** : remplace par l'URL de ta page WordPress (utilisée dans les partages).
- **`CONFIG.hashtags`** : hashtags du tweet.
- **`CONFIG.votesUrl`** : endpoint Google Apps Script qui collecte les votes et renvoie les
  statistiques (garder/vendre par joueur). Laisser vide pour désactiver la fonctionnalité
  « Avis de la communauté ». Les votes sont écrits en `POST` (fire-and-forget) et lus en JSONP ;
  les données atterrissent dans le Google Sheet associé (onglets `Votes` bruts + `Tally` agrégé).
- **`PLAYERS`** : tableau de l'effectif. Le champ **`val`** = valeur marchande estimée en **M€**
  → modifie-le librement (ex : `"val": 28`). Tu peux aussi retirer/ajouter un joueur.
- **`LOAN`** : joueurs **prêtés** (n'appartiennent pas à l'OM). `nom -> option d'achat (M€)` ou
  `null` si pas d'option. Un prêté **vendu/laissé rapporte 0 €** ; **gardé**, il coûte son option d'achat.
- **`EXPIRING`** : joueurs en **fin de contrat** (partent libres, rapportent 0 €).
- **`RETURNING`** : joueurs **de retour de prêt** (propriété de l'OM). Ils sont **vendables**
  (rapportent leur `val`) et s'ajoutent à l'effectif ; badge bleu « Retour de prêt ».

Le bilan final affiche : **Ventes (+)** − **Options d'achat (−)** = **Solde net**.

## Mettre à jour l'effectif (optionnel)

L'effectif est figé au moment de la génération. Pour le rafraîchir, ré-appelle l'API Sportmonks :

```
GET https://api.sportmonks.com/v3/football/teams/44?api_token=TON_TOKEN&include=players.player.position;players.player.detailedPosition
```

(`44` = ID de l'Olympique de Marseille.)

---
*Données : Sportmonks. Valeurs marchandes estimées, indicatives. Outil non officiel.*
