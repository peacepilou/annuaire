# Annuaire

Aujourd'hui, tu refais tout le module Angular d'un coup, et seul.

Ta mission : un annuaire de 30 fausses personnes, servies par l'API [randomuser.me](https://randomuser.me). Une liste en cartes, une fiche par personne, une recherche et des favoris. Et tu pars de zéro : même le `ng new`, c'est toi :)

## Démarrer

Ce dépôt te donne deux choses : ce README, et `people-backup.json`, la roue de secours (on en parle juste après).

Clone-le sous le nom `annuaire-sujet` :

```bash
git clone https://github.com/peacepilou/annuaire.git annuaire-sujet
```

Ton app, tu la crées **à côté** de ce dossier, pas dedans. Tu te souviens pourquoi ? Dans un dossier déjà suivi par Git, `ng new` saute la création du dépôt.

### Avant de commencer

Ouvre l'adresse de l'API dans ton navigateur :

```text
https://randomuser.me/api/?seed=cda2&results=30&nat=fr
```

Tu vois un gros objet JSON. Prends cinq minutes pour le lire : tout le TP repose dessus. Le `seed=cda2` fait que toute la classe reçoit les **mêmes** 30 personnes, à chaque appel.

### Si l'API tombe

Ça arrive. Le fichier `people-backup.json` contient exactement la même réponse, tirée avec le même `seed`.

1. Copie `people-backup.json` dans le dossier `public/` de ton app (tu le fais à l'étape 2).
2. Dans ton service, remplace l'adresse de l'API par `/people-backup.json`.

Même forme, mêmes personnes : le reste de ton code ne bouge pas.

## Les règles

- **Avance étape par étape.** Chaque étape sert de fondation à la suivante.
- **Un commit par étape**, du genre `step 3: show the people as cards`. À la fin, tu compares avec la correction, commit par commit.
- **Bloqué ?** Chaque étape a un indice replié. Ouvre-le après avoir cherché un bon moment, pas avant.
- **C'est toi qui écris le code.** Tes cours et [angular.dev](https://angular.dev) sont ouverts, ton IA reste fermée : aujourd'hui, on regarde ce que **toi** tu sais faire.

Et la règle d'or du jour : **la liste se charge une seule fois, dans le service.** Toute la classe passe par la même connexion : chaque élève appelle l'API une fois, au démarrage de son app.

---

## Le socle : étapes 1 à 6

### Étape 1 · Créer l'app

Dans ton dossier de formation :

```bash
ng new annuaire --ai-config none
```

Aux questions du terminal, réponds comme au cours : `CSS`, et `N` pour le rendu côté serveur.

Puis vide `app.html` : la page de démo d'Angular laisse la place à un en-tête avec le titre « Annuaire », et à `<router-outlet />` juste en dessous. Lance `ng serve`.

**Ce que tu dois voir :** sur `http://localhost:4200`, le titre « Annuaire », et plus rien de la page d'Angular.

<details>
<summary>Un indice</summary>

`app.html` fait 20 000 caractères, mais tu peux tout effacer. Garde seulement `<router-outlet />` : c'est là que tes pages vont s'afficher à partir de l'étape 3.
</details>

### Étape 2 · Le service qui charge les 30 personnes

D'abord, les types. Dans `src/app/models/person.model.ts`, décris :

- `Person` : une personne, avec seulement les champs que tu vas afficher. `login.uuid`, `gender`, `name.first` et `name.last`, `email`, `phone`, `cell`, `dob.age`, `location.city` et `location.country`, `picture.large`.
- `PeopleResponse` : la réponse entière de l'API. Regarde bien où est rangé le tableau.

Ensuite, le service :

```bash
ng generate service people --type=service
```

`PeopleService` charge les 30 personnes avec `HttpClient`, et les garde dans un signal `people`. Deux autres signaux racontent où on en est : `loading` (on attend) et `error` (ça a raté).

Pour vérifier, affiche le nombre de personnes sous le titre, dans `app.html`.

Et pense à copier `people-backup.json` dans `public/` maintenant : le jour où l'API tombe, tu seras content de l'avoir.

**Ce que tu dois voir :** « 30 personnes » sous le titre. Dans l'onglet Réseau des outils de développement, **une seule** requête vers `randomuser.me`.

<details>
<summary>Un indice</summary>

Un service n'est fabriqué qu'une fois, puis partagé par tous ceux qui l'injectent. Lance donc le chargement depuis son `constructor` : il ne partira qu'une fois, quelle que soit la page.

Et si la console du navigateur affiche `NullInjectorError` avec `HttpClient`, c'est qu'il manque `provideHttpClient()` dans `app.config.ts`.
</details>

### Étape 3 · La liste en cartes

Deux composants :

```bash
ng generate component person-card
ng generate component people-list
```

`PersonCard` reçoit une personne par un `input` obligatoire, et affiche sa photo, son prénom et son nom, et sa ville. `PeopleList` est la page d'accueil (la route vide) : elle affiche une carte par personne avec `@for`.

Pendant le chargement, la page affiche « Chargement de l'annuaire… ». Si l'appel rate : « L'annuaire est indisponible. Réessaie dans un instant. »

| Ce que tu vérifies                                           | Résultat attendu                                         |
| ------------------------------------------------------------ | -------------------------------------------------------- |
| La page d'accueil                                            | 30 cartes, avec photo, nom et ville                      |
| La première carte                                            | Logan Fleury, Aubervilliers                              |
| Onglet Réseau en « Slow 4G », puis recharge                  | « Chargement de l'annuaire… », puis les cartes           |
| Dans le service, `randomuser.me` devient `randomuser.mee`    | Le message d'erreur, et aucune carte (remets l'adresse !) |

<details>
<summary>Un indice</summary>

Le trio `@if (…loading())` / `@else if (…error(); as message)` / `@else` du chapitre sur HttpClient fait tout le travail. Pour le `track` du `@for`, il te faut une valeur unique par personne : `person.login.uuid`.
</details>

### Étape 4 · La fiche d'une personne

Un clic sur une carte ouvre `/personnes/:id`, où `:id` est le `login.uuid` de la personne. La fiche `PersonDetail` affiche sa photo, son nom, son âge, son email, son téléphone, son portable et sa ville, avec un lien « ← Retour à l'annuaire ».

Piège : randomuser.me n'a **pas** d'adresse pour une seule personne. Pas de nouvel appel, donc. La liste, ton service l'a déjà.

| Ce que tu vérifies                              | Résultat attendu                                                              |
| ----------------------------------------------- | ----------------------------------------------------------------------------- |
| Un clic sur Eléonore Legrand                    | L'adresse devient `/personnes/1c36fcb0-…`, avec eleonore.legrand@example.com, Orléans |
| Retour, puis un clic sur une autre carte        | Sa fiche, et aucune nouvelle requête vers `randomuser.me` dans l'onglet Réseau |
| F5 sur une fiche                                | La fiche revient, après « Chargement de l'annuaire… »                         |
| `http://localhost:4200/personnes/n-existe-pas`  | « Personne introuvable »                                                      |

<details>
<summary>Un indice</summary>

`withComponentInputBinding()` dans `app.config.ts`, et le `:id` arrive tout seul dans un `input` nommé `id`. Ensuite, un `computed` qui fait un `find` dans `people()`, comme la fiche produit du cours.

Après un F5, la liste est vide le temps du chargement : teste `loading()` **avant** de dire « introuvable ».
</details>

### Étape 5 · La recherche

Au-dessus des cartes, un champ « Rechercher ». La liste se filtre à chaque lettre tapée, sur le prénom et le nom, sans tenir compte des majuscules.

La liste affichée se **calcule**, à partir de la liste complète et du texte tapé.

| Tu tapes      | Résultat attendu                                       |
| ------------- | ------------------------------------------------------ |
| `ben`         | 2 cartes : Benjamin Fabre, Benjamin Riviere            |
| `BEN`         | Les deux mêmes                                         |
| `fabre`       | 2 cartes : Benjamin Fabre, Noham Fabre                 |
| `benjamin f`  | 1 carte : Benjamin Fabre                               |
| `xyz`         | « Aucune personne ne correspond à ta recherche. »      |
| Champ vidé    | Les 30 cartes reviennent                               |

<details>
<summary>Un indice</summary>

Le champ, c'est un Signal Form d'un seul champ : `filters = signal({ search: '' })`, puis `form(this.filters)`, et `[formField]` sur l'`input`. Le filtre, c'est un `computed` qui lit `this.filters().search` et `people()`. `toLowerCase()` des deux côtés, et `@empty` pour le message.
</details>

### Étape 6 · Les favoris

Chaque carte a une étoile : ☆ vide, ★ pleine. Un clic l'allume ou l'éteint, et on reste sur la liste.

La carte se contente de **prévenir** son parent avec un `output`, et reçoit par un `input` l'info « je suis en favori ». Les favoris vivent dans un `FavoritesService`, qui les range dans `localStorage` à chaque changement et les relit au démarrage.

Une page `/favoris` affiche les mêmes cartes, mais seulement les favoris. Dans l'en-tête, deux liens : « Annuaire » et « Favoris (2) », avec le nombre à jour.

| Ce que tu vérifies                                         | Résultat attendu                                               |
| ---------------------------------------------------------- | -------------------------------------------------------------- |
| Clic sur l'étoile de Louane Adam                           | ★, « Favoris (1) », et tu restes sur la liste                   |
| Étoile sur Kenzo Marie, puis « Favoris »                   | Les cartes de Louane Adam et Kenzo Marie                        |
| F5 sur `/favoris`                                          | Toujours les deux                                               |
| Clic sur ★ depuis `/favoris`                               | La carte disparaît de la page                                   |
| Plus aucun favori sur `/favoris`                           | Un message : aucun favori pour l'instant                        |
| Application › Local Storage                                | Une clé `favorites`, avec un tableau d'uuid                     |
| Étoile sur Ethan Clement trouvé par `ethan`, puis champ vidé | Son étoile est toujours pleine                                |

<details>
<summary>Un indice</summary>

Le bouton étoile ne doit pas être **dans** le lien `<a>` de la carte : sinon, chaque clic sur l'étoile ouvre aussi la fiche.

`localStorage` ne range que du texte : `JSON.stringify` pour écrire le tableau, `JSON.parse` pour le relire. Et le tout premier jour, `localStorage.getItem` renvoie `null` : `?? '[]'` te sauve.
</details>

---

## Les bonus

Le socle tourne ? Choisis ceux qui te tentent, dans l'ordre que tu veux.

### Bonus · La pagination

10 cartes par page, avec « Précédent », « Suivant » et « Page 1 / 3 ».

| Ce que tu vérifies                       | Résultat attendu                                   |
| ---------------------------------------- | -------------------------------------------------- |
| La première page                         | 10 cartes, « Précédent » désactivé                 |
| Deux clics sur « Suivant »               | « Page 3 / 3 », « Suivant » désactivé              |
| `ben` tapé depuis la page 3              | « Page 1 / 1 » et les deux Benjamin                |

<details>
<summary>Un indice</summary>

Un signal `page`, et deux `computed` : le nombre de pages, et la tranche de la liste filtrée avec `slice`. Chaque frappe dans la recherche remet `page` à 1 : un `(input)` sur le champ suffit.
</details>

### Bonus · Le filtre par genre

Une liste déroulante « Tous / Femmes / Hommes », qui se combine avec la recherche.

| Tu choisis                | Résultat attendu                                      |
| ------------------------- | ----------------------------------------------------- |
| Femmes                    | 12 personnes                                          |
| Hommes                    | 18 personnes                                          |
| Hommes, et `ben` tapé     | Les deux Benjamin                                     |
| Femmes, et `ben` tapé     | « Aucune personne ne correspond à ta recherche. »     |

<details>
<summary>Un indice</summary>

Ajoute `gender: 'all'` au signal de l'étape 5 : `[formField]` marche aussi sur un `<select>`. Le `computed` gagne un deuxième `filter`.
</details>

### Bonus · Ajouter un contact

Une page `/ajouter`, avec un formulaire validé par Signal Forms : prénom (obligatoire, 2 caractères au moins), nom (obligatoire), genre, âge (entre 1 et 120), email (obligatoire, au bon format), téléphone (facultatif), ville (obligatoire).

Une fois envoyé, le contact s'ajoute en tête de l'annuaire, et tu arrives sur sa fiche. Comme photo, prends `https://randomuser.me/api/portraits/lego/1.jpg`.

| Ce que tu vérifies                         | Résultat attendu                                              |
| ------------------------------------------ | ------------------------------------------------------------- |
| « Ajouter » avec le formulaire vide        | Un message sous chaque champ obligatoire, et on reste là      |
| Prénom `A`, email `pas-un-email`           | « 2 caractères au moins. », et l'email refusé                 |
| Un contact valide                          | Sa fiche s'ouvre, « 31 personnes », sa carte en premier       |
| F5                                         | Il a disparu : « 30 personnes »                               |

La dernière ligne est normale : randomuser.me n'enregistre rien, et ton contact ne vit que dans le signal du service, le temps que la page reste ouverte.

<details>
<summary>Un indice</summary>

`email()` fait partie des règles de `@angular/forms/signals`, comme `required()` et `min()`. L'uuid, tu le fabriques avec `crypto.randomUUID()`. Une méthode `add()` dans `PeopleService`, puis `router.navigateByUrl()` vers la fiche.
</details>

### Bonus · Un guard sur les favoris

Sans aucun favori, `/favoris` ne sert à rien : un guard renvoie vers l'annuaire.

```bash
ng generate guard favorites
```

Le CLI te demande quel type de guard : choisis `CanActivate`.

| Ce que tu vérifies                          | Résultat attendu                       |
| ------------------------------------------- | -------------------------------------- |
| Aucun favori, `/favoris` tapé dans l'adresse | Retour sur l'annuaire                  |
| Un favori, puis « Favoris »                 | La page s'ouvre                        |

<details>
<summary>Un indice</summary>

C'est le guard du chapitre sur le Router, avec une autre condition : `inject(FavoritesService)` à la place de la connexion, et `parseUrl('/')` pour la redirection.
</details>

---

Bon courage, et pense à tes commits !
