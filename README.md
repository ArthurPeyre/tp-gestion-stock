# TD JavaScript

## Contexte du projet

Une entreprise a besoin d'un outil de gestion de ses stocks. L'ajout de produit au stock, leur modification et suppression sera les principales fonctionnalités à intégrer. Quelques fonctionnalités supplémentaires seront à rajouter en suivant, telles que la recherce de produit, et certaine fonction d'UX qui agiront principalement sur le style et classes CSS.

Pour ce projet, les code HTML et CSS sont déjà entièrement fournis. Vous aurez seulement le JavaScript à compléter afin de rendre l'application fonctionnelle.

## Étape 1 - Prendre connaissance du code fourni

Vous pouvez observer la structure HTML, avec les objets et leurs classes et identifiant.

Pour ce qui est du code JS, vous verrez du code déjà fourni. Ce sont les variables que vous utiliserez dans le code, ou qui sont nécessaire au bon fonctionnement de l'application.

### Aide au développement - Visual Studio Code

Si vous utilisez VSCode, vous pouvez installer l'extension "Live Server" qui vous permettra de visualiser le résultats de toutes modifications du code instantanément dans votre navigateur, au lieu de rafraîchir manuellement la page.

Pour lancer l'extension, clique droit sur le code de votre page HTML, puis choisir l'option "Open with Live Server". Cela devrait vous ouvrir automatiquement une nouvelle page web.

## Étape 2 - Création de la classe produit

### Signature de la classe

Dans un premier temps, vous devrez créer une classe Produit, qui répondra à ces critères :
- Le nom de la classe : "Product"
- Un objet a les propriétés suivantes:
    - id : Identifiant unique
    - name : Nom du produit
    - description : _Vous n'êtes pas bête..._
    - brand : Marque du produit
    - stock : Quantité du produit en stock
- Seul le nom, la description, la marque et le stock du produit sont à valoriser lors de la création d'un objet produit.

### Génération de l'identifiant unique

Aujourd'hui, rare seront les bases de données dans lesquelles les identifiants sont générés de cette façon : 1, 2, 3, ... Il est beaucoup plus courant maintenant de croiser ce genre d'identifiant : "e114e88e-2ed2-4978-8967-ceec811583fe".
Bien évidemment, la génération se ferait par la base de données, et jamais du côté du client, mais pour ce TD nous simuleront la génération de cette identifiant lors de la création de l'objet.

Pour cela, vous utiliserai la fonction ``crypto.randomUUID()`` pour valoriser la variable 'id' de l'objet Product.

### Passage au HTML

Vous rajouterez la propriété suivant à votre classe :

```js
this.toHTML = () => {
    return `<div class="row"><div class="data"><span>${this.name}</span><span>${this.description}</span><span>${this.brand}</span><span class="${ this.stock > 0 ? "positive" : this.stock < 0 ? "negative" : "" }">${ this.stock  }</span></div><div class="actions ${ this.id === selectedProductID ? "selected" : "" }"><button onclick="setEditForm('${this.id}')">${svgEdit}</button><button class="delete" onclick="removeProduct('${this.id}')">${svgDel}</button></div>`;
};
```

Cette propriété est en réalité une fonction renvoyant le code HTML ayant la bonne structure, avec les données de son objet. Elle sera appelée à l'étape 4, lors de l'affichage des produits.

### Aide au développement

Pour vous aider, vous pouvez observer le code de la fonction ``initProducts()`` qui simule l'appel à l'API afin de récupérer un échantillon de produit.

## Étape 3 - Liaison des variables globales aux éléments du DOM

Dans cette étape, nous lierons les variables globales aux éléments du DOM du DOM, afin de pouvoir les manipuler à l'aide de JavaScript.

### Vérification que le contenu du DOM est chargé

Il est important de comprendre que le code JavaScript s'exécute en parallèle du chargement de la page. Cela signifie que parfois, le code JavaScript est exécuté avant que tous les éléments de la page, et donc du DOM, ne soit générés. Cela peut donc entraîner des erreurs, par exemple lors d'une tentative de récupération d'un objet du DOM. Si l'élément n'est pas encore chargé, alors la variable ne sera pas correctement valorisée.

C'est pourquoi il existe une fonction permettant d'exécuter un morceau de code uniquement lorsque le DOM est entièrement chargé. Cette méthode est la suivante :

```js
window.addEventListener('DOMContentLoaded', () => {
    // ...
});
```

La méthode ``addEventListener()`` est un écouteur d'évènements, qui prend en paramètre ce qu'elle doit écouter (ici, ``'DOMContentLoaded'``), et ce qu'elle doit exécuter. Ici, l'évènement signifie qu'elle écoute que le contenu du DOM est chargé. Elle est rataché à l'objet ``window`` qui est la référence à votre page HTML.

### Valorisation des variables avec les éléments du DOM

Vous ajouterez le code ci-dessous, qui doit s'exécuter une fois que le DOM est chargé:

```js
// HTML Elements
productsContainer = document.getElementById('products');

newInputsForm.name = document.getElementById('newnametxt');
newInputsForm.description = document.getElementById('newdesctxt');
newInputsForm.brand = document.getElementById('newbrandtxt');
newInputsForm.stock = document.getElementById('newstocknum');
addProductBtn = document.getElementById('addProductBtn');

editInputsForm.name = document.getElementById('editnametxt');
editInputsForm.description = document.getElementById('editdesctxt');
editInputsForm.brand = document.getElementById('editbrandtxt');
editInputsForm.stock = document.getElementById('editstocknum');
editProductBtn = document.getElementById('editProductBtn');

filterInput = document.getElementById('filter');

formsContainer = document.getElementById('forms__container');
forms = document.querySelectorAll('.forms > *');

toggleFormsBtn = document.getElementById('toggleFormsContainer');
toggleFormsBtn.addEventListener('click', () => {
    formsContainer.classList.toggle('hidden');
});
```

### Initialisation des données

À la suite de ce code, appeler la methode ``initProducts()`` afin de valoriser la variable ``products``.

### Aide au développement - Les différents évènements

Il existe un nombre important d'évènement avec des utilités propres à chacun. En voici une liste non exhaustive :

| Évènement | À quoi il sert |
| --------- | --------- |
| ``DOMContentLoaded`` | Se déclenche quand le HTML est entièrement chargé et analysé (sans attendre images, CSS, etc.) |
| ``click`` | Déclenché lors d’un clic de souris |
| ``submit`` | Lors de la soumission d’un formulaire |
| ``input`` | Quand la valeur d’un champ change en temps réel |
| ``scroll`` | Quand l’utilisateur fait défiler la page ou un élément |

## Étape 4 - Affichage des produits

Dans cette étape, vous allez devoir coder une méthode actualisant la liste de produits.

### Création de la métode

On nommera cette méthode ``refreshProducts()``

Cette méthode doit : 
1. Réinitialiser le contenu de l'élément HTML contenant les produits (``productsContainer``) à vide.
2. Vérifier que la liste des produits est supérieure à 0.
3. Si vrai, appeler la fonction ``toHTML`` de chacun des produits de la liste, et les ajouter au contenu HTML.
3. Si faux, valoriser le contenu HTML à l'aide de la variable ``emptyProducts`` préalablement valoriser.

### Appel de la méthode

Maintenant que le rafraîchissement des produits est opérationnel, il ne reste plus qu'à appeler la méthode lorsque le DOM est chargé.

Vous devriez maintenant voir s'afficher les produits sur votre application.

## Étape 5 - Création de nouveaux produits

Dans cette étape, vous allez développer la fonctionnalité d'ajout d'un nouveau produit via le formulaire dédié.

### La méthode

Vous allez intégrer une nouvelle méthode nommée ``addProduct()`` qui :
1. Récupère les données saisies dans le formulaire.
2. Vérifie que tous les champs sont remplis.
3. Si vrai :
    - Créer l'objet avec les données saisies.
    - Ajouter l'objet à la liste d'objet ``products``.
    - Rafraîchir le contenu HTML pour y voir afficher le nouveau produit.
    - Vider tous les champs du formulaire.
4. Si faux, alerter l'utilisateur avec le message ``"Tous les champs sont requis"``.

### Lier le clic à la méthode

Maintenant que la méthode est prête, vous pouvez ajouter un écouteur d'évènement lors du clic du bouton de soumission de ce formulaire.

### **Attention !**
En JavaScript, si vous appelez une fonction en ajoutant les parentèses comme ceci : ``addProduct()``, la fonction sera exécuté. Cependant, dans un écouteur d'évènement, nous ne souhaitons pas appeler la méthode a exécuter lors de l'instanciation de l'évènement, donc il faut retirer les parenthèses et simplement garder le nom de la méthode : ``addProduct``.

### Aide au développement

| Signature | Utilité |
| --------- | ------- |
| ``.value`` | Atteindre la valeur de l'input |
| ``.trim()`` | Supprimer les espaces inutiles au début et à la fin d’une chaîne de caractères. Permet de vérifier notamment si la chaîne saisir n'est pas remplie uniquement d'espace. |
| ``.push()`` | Ajouter à la fin de la liste un nouvel item |
| ``window.alert()`` | Créer une alerte dans l'application qui affichera le texte passé en paramètre. |

## Étape 6 - Suppression d'un produit

Dans cette étape, vous allez développer la fonctionnalité de suprression d'un produit via le bouton dédié à chacune des lignes des produits.

Notez que son appel est déjà effectué dans la propriété ``toHTML()`` de chaque produit.

### La méthode

Vous allez intégrer la méthode ``removeProduct`` qui :
1. Prend l'identifiant du produit en paramètre.
2. Modifier la liste de produits ``products`` afin de garder tous les produits, sauf celui qui a pour identifiant celui passé en paramètre.
3. Rafraîchir le contenu HTML avec les nouvelles données

### Aide au développement

| Signature | Utilité | Usage |
| --------- | ------- | ----- |
| ``.filter`` | Renvoie la liste associé filtré selon la condition souhaité | ``array.filter(item => condition)`` |

## Étape 7 - Modification d'un produit

Dans la continuité, vous allez intégrer la fonctionnalité d'édition d'un produit. Il sera notamment possible de modifier l'intégralité de ses données, à l'exception de son identidiant.

### Valorisation du formulaire dédié

Tout d'abord, il faudra créer une méthode qui valorisera les champs de saisie du formulaire de modification avec les données du produit sélectionné. Pour cela, vous stockerez l'identifiant du produit dans la variable ``selectedProductID``. Son stockage sera établie dans la méthode suivante.

1. La méthode doit être sous forme d'une constante, nommée ``resetEditForm``.
2. Elle récupère l'objet produit de la liste ``products`` ayant l'``'id'`` correspondant à l'identifiant sélectionné.
3. Si l'objet est ``null``, réinitialise la variable ``selectedProductID``, vide le formulaire dédié, et stop la fonction.
4. Sinon, valoriser les champs du formulaire avec les données de l'objet.

### Gestion de l'affichage du formulaire

La méthode ``setEditForm`` :
0. Intégrer au début de la fonction:
```js
if (id === selectedProductID) {
    selectedProductID = '';

    formsContainer.classList.add('hidden');

    forms.forEach(form => {
        form.classList.toggle('hidden');
    });

    refreshProducts();
    resetEditForm();

    return;
}
```
1. Prend en paramètre l'identifiant du produit.
2. Valorise ``selectedProductID`` avec le paramètre.
3. Rafraîchi le contenu HTML _(Gestion du style)_.
4. Appel de la fonction précèdente ``resetEditForm``.
5. Rajouter à la suite :
```js
// Si formulaire d'édition caché, inverser la classe 'hidden' pour chacun des deux formulaires.
if (forms[1].classList.contains('hidden')) {
    forms.forEach(form => {
        form.classList.toggle('hidden');
    });
}
```

Cette fonction est déjà appelé, tout comme ``removeProduct``, dans la propriété ``toHTML()`` de chaque produit.

### Édition du produit

Pour finir avec cette fonctionnalité, il faudra créer une dernière fonction ``editProduct()`` qui récupèrera les nouvelles données modifiées du formulaire, et remplacera celles de l'objet.

1. Récupérer les valeurs des champs du formulaire en vérifiant que leur contenu est correct.
2. Récupérer l'objet à l'aide de la variable ``selectedProductID``.
3. Si l'objet est trouvé, assigner à l'objet les nouvelles valeurs.
4. Sinon, alerter l'utilisateur que le produit à modifier est introuvable.
5. Vider le formulaire.
6. Supprimer la valeur de l'identifiant stocké.
7. Rafraîchir le contenu HTML à l'aide de ``refreshProducts()``.

Vous pouvez dès à présent ajouter à ``editProductBtn`` un écouteur d'évènement qui exécutera cette fonction.

N'oubliez pas également l'écouteur d'évènement sur le bouton d'annulation des modification qui appellera la fonction ``resetEditForm``. Vous devrez par conséquent récupérer vous même l'élément HTML à partir du DOM.

### Aide au développement

| Signature | Utilité | Usage |
| --------- | ------- | ----- |
| ``.find`` | Renvoie l'objet associé filtré selon la condition souhaité | ``array.find(item => condition)`` |

## Filtrer les produits

Dans cette étape, vous intégrerez une dernière fonctionnalité, qui permet de filtrer les produits à afficher selon ce qui est saisi dans le champ de recherche.

### La méthode

Pour cela, vous devrez :
1. Créer la constante ``filterProducts`` qui sera utilisé comme fonction.
2. Récupérer la valeur du champ recherche atteignable depuis la variable ``filterInput``.
3. Revaloriser ``filteredProducts`` qui est, au lancement de la page, identique à ``products``:
    - Si value n'est pas ``null``, filtrer ``products`` de sorte à ne renvoyer que les produits répondant aux critères suivant dans l'ordre hiérarchique:
        1. Nom
        2. Marque
        3. Stock
    - Sinon, lui redonner le contenu de ``products``.

### Application du filtre

Appeler la fonction dans la méthode ``refreshProducts()`` et adapter son contenu en conséquence.

Appeler la bonne méthode avec bon écouteur d'évènement au bon élément... **Bonne chance !**