Views (Vues)
############
.. php:namespace:: Cake\View

.. php:class:: View

Les Views (Vues) sont le **V** dans MVC. Les vues sont chargées de générer la
sortie spécifique requise par la requête. Souvent, cela est fait sous forme
HTML, XML ou JSON, mais le streaming de fichiers et la création de PDFs que les
utilisateurs peuvent télécharger sont aussi de la responsabilité de la
couche View.

CakePHP a quelques classes de vue déjà construites pour gérer les scénarios de
rendu les plus communs:

- Pour créer des services web XML ou JSON, vous pouvez utiliser
  :doc:`views/json-and-xml-views`.
- Pour servir des fichiers protégés, ou générer des fichiers dynamiquement,
  vous pouvez utiliser :ref:`cake-response-file`.
- Pour créer plusieurs vues pour un thème, vous pouvez utiliser
  :doc:`views/themes`.

The App View
============

``AppView`` est la classe View par défaut de votre application. ``AppView``
étend lui même la classe ``Cake\View\View`` de CakePHP et est définie dans
``src/View/AppView.php`` comme suit:

.. code-block:: php

    namespace App\View;

    use Cake\View\View;

    class AppView extends View
    {
    }

Vous pouvez utiliser ``AppView`` pour charger des helpers qui seront utilisés
dans toutes les vues rendues de votre application. CakePHP fournit une méthode
``initialize()`` qui est invoquée à la fin du constructeur de la View pour ce
type d'utilisation:

.. code-block:: php

    namespace App\View;

    use Cake\View\View;

    class AppView extends View
    {

        public function initialize()
        {
            // Toujours activer le helper MyUtils
            $this->loadHelper('MyUtils');
        }

    }

Templates de Vues
=================

La couche view de CakePHP c'est la façon dont vous parlez à vos utilisateurs.
La plupart du temps, vos vues afficheront des documents (X)HTML pour les
navigateurs, mais vous pourriez aussi avoir besoin de fournir des données AMF
à un objet Flash, répondre à une application distante via SOAP ou produire un
fichier CSV pour un utilisateur.

Les fichiers de template de CakePHP sont écrits en pur PHP et ont par défaut .ctp
(Cakephp TemPlate) comme extension. Ces fichiers contiennent toute la logique
de présentation nécessaire à l'organisation des données reçues du controller,
dans un format qui satisfasse l'audience que vous recherchez. Si vous préférez
utiliser un langage de template comme Twig, ou Smarty, une sous-classe de View
fera le pont entre votre langage de template et CakePHP.

Un fichier de template est stocké dans ``src/Template/``, dans un sous-dossier
portant le nom du controller qui utilise ce fichier. Il a un nom de fichier
correspondant à son action. Par exemple, le fichier de vue pour l'action
"view()" du controller Products devra normalement se trouver dans
``src/Template/Products/view.ctp``.

La couche vue de CakePHP peut être constituée d'un certain nombre de parties
différentes. Chaque partie a différents usages qui seront présentés dans ce
chapitre :

- **views**: Les Views sont la partie de la page qui est unique pour l'action
  lancée. Elles sont la substance de la réponse de votre application.
- **elements** : morceaux de code de view plus petits, réutilisables. Les
  éléments sont habituellement rendus dans les vues.
- **layouts** : fichiers de template contenant le code de présentation qui se
  retrouve dans plusieurs interfaces de votre application. La plupart des
  vues sont rendues à l'intérieur d'un layout.
- **helpers** : ces classes encapsulent la logique de vue qui est requise
  à de nombreux endroits de la couche view. Parmi d'autres choses, les helpers
  de CakePHP peuvent vous aider à créer des formulaires, des fonctionnalités
  AJAX, à paginer les données du model ou à délivrer des flux RSS.
- **cells**: Ces classes fournissent des fonctionnalités de type controller
  en miniature pour créer des components avec une UI indépendante. Regardez
  la documentation :doc:`/views/cells` pour plus d'informations.

Variables de Vue
----------------

Toute variable que vous définissez dans votre controller avec ``set()`` sera disponible à la fois dans la vue et dans le layout que votre action utilise.
En plus, toute variable définie sera aussi disponible dans tout element.
Si vous avez besoin de passer des variables supplémentaires de la
vue vers le layout, vous pouvez soit appeler ``set()`` dans le template de vue,
soit utiliser un :ref:`view-blocks`.

Vous devriez vous rappeler de **toujours** échapper les données d'utilisateur
avant de les afficher puisque CakePHP n'échappe automatiquement la sortie. Vous
pouvez échapper le contenu d'utilisateur avec la fonction ``h()``::

    <?= h($user->bio); ?>

Définir les Variables de Vue
----------------------------

.. php:method:: set(string $var, mixed $value)

Les vues ont une méthode ``set()`` qui fonctionne de la même façon que
``set()`` qui se trouve dans les objets Controller. Utiliser set() à
partir de la vue va ajouter les variables au layout et aux elements qui seront
affichés plus tard. Regardez :ref:`setting-view_variables` pour plus
d'informations sur l'utilisation de ``set()``.

Dans votre fichier de vue, vous pouvez faire::

    $this->set('activeMenuButton', 'posts');

Ensuite, dans votre layout, la variable ``$activeMenuButton`` sera disponible
et contiendra la valeur 'posts'.

.. _extending-views:

Vues étendues
-------------

Une vue étendue vous permet d'encapsuler une vue dans une autre. En combinant
cela avec :ref:`view blocks <view-blocks>`, cela vous donne une façon puissante
pour garder vos vues :term:`DRY`. Par exemple, votre application a une sidebar
qui a besoin de changer selon la vue spécifique en train d'être rendue. En
étendant un fichier de vue commun, vous pouvez éviter de répéter la balise
commune pour votre sidebar, et seulement définir les parties qui changent:

.. code-block:: php

    <!-- src/Template/Common/view.ctp -->
    <h1><?= $this->fetch('title') ?></h1>
    <?= $this->fetch('content') ?>

    <div class="actions">
        <h3>Related actions</h3>
        <ul>
        <?= $this->fetch('sidebar') ?>
        </ul>
    </div>

Le fichier de vue ci-dessus peut être utilisé comme une vue parente. Il
s'attend à ce que la vue l'étendant définisse des blocks ``sidebar``
et ``title``. Le block ``content`` est un block spécial que CakePHP
crée. Il contiendra tous les contenus non capturés de la vue étendue.
En admettant que notre fichier de vue a une variable ``$post`` avec les
données sur notre post. Notre vue pourrait ressembler à ceci:

.. code-block:: php

    <!-- src/Template/Posts/view.ctp -->
    <?php
    $this->extend('/Common/view');

    $this->assign('title', $post);

    $this->start('sidebar');
    ?>
    <li>
    <?php
    echo $this->Html->link('edit', [
        'action' => 'edit',
        $post['Post']['id']
    ]); ?>
    </li>
    <?php $this->end(); ?>

    // The remaining content will be available as the 'content' block
    // In the parent view.
    <?= h($post['Post']['body']) ?>

L'exemple ci-dessus vous montre comment vous pouvez étendre une vue, et
remplir un ensemble de blocks. Tout contenu qui ne serait pas déjà dans un block
défini, sera capturé et placé dans un block spécial appelé ``content``. Quand
une vue contient un appel vers un ``extend()``, l'exécution continue jusqu'à la
fin de la vue actuelle. Une fois terminé, la vue étendue va être générée. En
appelant ``extend()`` plus d'une fois dans un fichier de vue, le dernier appel
va outrepasser les précédents::

    $this->extend('/Common/view');
    $this->extend('/Common/index');

Le code précédent va définir ``/Common/index.ctp`` comme étant la vue parente
de la vue actuelle.

Vous pouvez imbriquer les vues autant que vous le voulez et que cela vous est
nécessaire. Chaque vue peut étendre une autre vue si vous le souhaitez. Chaque
vue parente va récupérer le contenu de la vue précédente en tant que block
``content``.

.. note::

    Vous devriez éviter d'utiliser ``content`` comme nom de block dans votre
    application. CakePHP l'utilise pour définir le contenu non-capturé pour
    les vues étendues.

Vous pouvez récupérer la liste de tous blocks existants en utilisant la méthode
``blocks()``::

    $list = $this->blocks();

.. _view-blocks:

Utiliser les Blocks de Vues
===========================

Les blocks de vue fournissent une API flexible qui vous permet de
définir des slots (emplacements), ou blocks, dans vos vues / layouts qui peuvent
être définies ailleurs. Par exemple, les blocks pour implémenter des choses
telles que les sidebars, ou des régions pour charger des ressources dans
l'en-tête / pied de page du layout. Un block peut être défini de deux manières.
Soit en tant que block capturant, soit en le déclarant explicitement. Les
méthodes ``start()``, ``append()``, ``prepend()``, ``assign()``, ``fetch()``
et ``end()`` vous permettent de travailler avec les blocks capturant::

    // Créer le block sidebar.
    $this->start('sidebar');
    echo $this->element('sidebar/recent_topics');
    echo $this->element('sidebar/recent_comments');
    $this->end();

    // Le rattacher à la sidebar plus tard.
    $this->start('sidebar');
    echo $this->fetch('sidebar');
    echo $this->element('sidebar/popular_topics');
    $this->end();

Vous pouvez aussi ajouter dans un block en utilisant ``append()``::

    $this->append('sidebar');
    echo $this->element('sidebar/popular_topics');
    $this->end();

    // Le même que ci-dessus.
    $this->append('sidebar', $this->element('sidebar/popular_topics'));

``assign()`` peut être utilisée pour nettoyer ou écraser un block en tout
temps::

    // Nettoyer le contenu précédent du block de sidebar
    $this->assign('sidebar', '');

Assigner le contenu d'un block est souvent utile lorsque vous voulez convertir
une variable de vue en un block. Par exemple, vous pourriez vouloir utiliser
un block pour le titre de la page et parfois le définir depuis le controller::

    // Dans une view ou un layout avant $this->fetch('title')
    $this->assign('title', $title);

La méthode ``prepend()`` a été ajoutée pour ajouter du contenu avant un block
existant::

    // Ajoutez avant la sidebar
    $this->prepend('sidebar', 'ce contenu va au-dessus de la sidebar');

.. note::

    Vous devriez éviter d'utiliser ``content`` comme nom de bloc. Celui-ci est
    utilisé par CakePHP en interne pour étendre les vues, et le contenu des
    vues dans le layout.

Afficher les Blocks
-------------------

Vous pouvez afficher les blocks en utilisant la méthode ``fetch()``. Cette
dernière va, de manière sécurisée, générer un block, en retournant '' si le
block n'existe pas::

    <?= $this->fetch('sidebar') ?>

Vous pouvez également utiliser fetch pour afficher du contenu, sous conditions,
qui va entourer un block existant. Ceci est très utile dans les layouts, ou
dans les vues étendues lorsque vous voulez, sous conditions, afficher des
en-têtes ou autres balises:

.. code-block:: php

    // dans src/Template/Layout/default.ctp
    <?php if ($this->fetch('menu')): ?>
    <div class="menu">
        <h3>Menu options</h3>
        <?= $this->fetch('menu') ?>
    </div>
    <?php endif; ?>

Vous pouvez aussi fournir une valeur par défaut pour un block
qui ne devrait pas avoir de contenu. Cela vous permet d'ajouter facilement
du contenu placeholder, pour des déclarations vides. Vous pouvez fournir
une valeur par défaut en utilisant le 2ème argument:

.. code-block:: php

    <div class="shopping-cart">
        <h3>Your Cart</h3>
        <?= $this->fetch('cart', 'Votre Caddie est vide') ?>
    </div>

Utiliser des Blocks pour les Fichiers de Script et les CSS
----------------------------------------------------------

:php:class:`HtmlHelper` est lié aux blocks de vue, et ses méthodes
:php:meth:`~HtmlHelper::script()`, :php:meth:`~HtmlHelper::css()`, et
:php:meth:`~HtmlHelper::meta()` mettent à jour chacun un block avec le même nom
quand il est utilisé avec l'option ``block = true``:

.. code-block:: php

    <?php
    // Dans votre fichier de vue
    $this->Html->script('carousel', ['block' => true]);
    $this->Html->css('carousel', null, ['block' => true]);
    ?>

    // Dans votre fichier de layout.
    <!DOCTYPE html>
    <html lang="en">
        <head>
        <title><?= $this->fetch('title') ?></title>
        <?= $this->fetch('script') ?>
        <?= $this->fetch('css') ?>
        </head>
        // reste du layout à la suite

Le :php:meth:`HtmlHelper` vous permet aussi de contrôler vers quels blocks vont
les scripts::

    // dans votre vue
    $this->Html->script('carousel', ['block' => 'scriptBottom']);

    // dans votre layout
    <?= $this->fetch('scriptBottom') ?>

.. _view-layouts:

Layouts
=======

Un layout contient le code de présentation qui entoure une vue.
Tout ce que vous voulez voir dans toutes vos vues devra être placé dans un
layout.

Le fichier de layout par défaut de CakePHP est placé dans
``src/Template/Layout/default.ctp``. Si vous voulez changer entièrement le
look de votre application, alors c'est le bon endroit pour commencer, parce que
le code de vue de rendu du controller est placé à l'intérieur du layout par
défaut quand la page est rendue.

Les autres fichiers de layout devront être placés dans ``src/Template/Layout``.
Quand vous créez un layout, vous devez dire à CakePHP où placer
la sortie pour vos vues. Pour ce faire, assurez-vous que votre layout contienne
``$this->fetch('content')``. Voici un exemple de ce à quoi un layout pourrait
ressembler:

.. code-block:: php

   <!DOCTYPE html>
   <html lang="en">
   <head>
   <title><?= $this->fetch('title'); ?></title>
   <link rel="shortcut icon" href="favicon.ico" type="image/x-icon">
   <!-- Include external files and scripts here (See HTML helper for more info.) -->
   <?php
   echo $this->fetch('meta');
   echo $this->fetch('css');
   echo $this->fetch('script');
   ?>
   </head>
   <body>

   <!-- Si vous voulez qu'un menu soit affiché pour toutes vos vues,
   incluez le ici -->
   <div id="header">
       <div id="menu">...</div>
   </div>

   <!-- C'est ici que je veux voir mes vues être affichées -->
   <?= $this->fetch('content') ?>

   <!-- Ajoute un footer pour chaque page affichée -->
   <div id="footer">...</div>

   </body>
   </html>

Les blocks ``script``, ``css`` et ``meta`` contiennent tout contenu défini
dans les vues en utilisant le helper HTML intégré. Il est utile pour inclure
les fichiers JavaScript et les CSS à partir des vues.

.. note::

    Quand vous utilisez ``HtmlHelper::css()`` ou
    ``HtmlHelper::script()`` dans les fichiers de template, spécifiez
    ``'block' => true`` pour placer la source html dans un
    block avec le même nom. (Regardez l'API pour plus de détails sur leur
    utilisation).

Le block ``content`` contient les contenus de la vue rendue.

Vous pouvez aussi définir le block ``title`` depuis l'intérieur
d'un fichier de vue::

    $this->assign('title', $titleContent);

Vous pouvez créer autant de layouts que vous souhaitez: placez les juste dans
le répertoire ``src/Template/Layout``, et passez de l'un à l'autre depuis les
actions de votre controller en utilisant la propriété
``$layout`` de votre controller ou de votre vue::

    // A partir d'un controller
    public function admin_view()
    {
        // stuff
        $this->layout = 'admin';
    }

    // A partir d'un fichier de vue
    $this->layout = 'loggedin';

Par exemple, si une section de mon site incorpore un plus petit espace pour
une bannière publicitaire, je peux créer un nouveau layout avec le plus
petit espace de publicité et le spécifier comme un layout pour toutes les
actions du controller en utilisant quelque chose comme::

    namespace App\Controller;

    class UsersController extends AppController
    {
        public function view_active()
        {
            $this->set('title', 'View Active Users');
            $this->layout = 'default_small_ad';
        }

        public function view_image()
        {
            $this->layout = 'image';
            // Output user image
        }
    }

Outre le layout par défaut, le squelette officiel d'application CakePHP dispose
également d'un layout 'ajax'. Le layout AJAX est pratique pour élaborer des
réponses AJAX - c'est un layout vide (la plupart des appels ajax ne nécessitent
qu'un peu de balise en retour, et pas une interface de rendu complète).

Le squelette d'application dispose également d'un layout par défaut pour aider
à générer du RSS.

Utiliser les layouts à partir de plugins
----------------------------------------

Si vous souhaitez utiliser un layout qui existe dans un plugin, vous pouvez
utiliser la :term:`syntaxe de plugin`. Par exemple pour utiliser le layout de
contact à partir du plugin Contacts::

    namespace App\Controller;

    class UsersController extends AppController
    {
        public function view_active()
        {
            $this->layout = 'Contacts.contact';
        }
    }


.. _view-elements:

Elements
========

.. php:method:: element(string $elementPath, array $data, array $options = [])

Beaucoup d'applications ont des petits blocks de code de présentation qui
doivent être répliqués d'une page à une autre, parfois à des endroits
différents dans le layout. CakePHP peut vous aider à répéter des parties
de votre site web qui doivent être réutilisées. Ces parties réutilisables
sont appelées des Elements. Les publicités, les boites d'aides, les contrôles
de navigation, les menus supplémentaires, les formulaires de connexion et de
sortie sont souvent intégrés dans CakePHP en elements. Un element est tout
bêtement une mini-vue qui peut être inclue dans d'autres vues, dans les
layouts, et même dans d'autres elements. Les elements peuvent être utilisés
pour rendre une vue plus lisible, en plaçant le rendu d'éléments répétitifs
dans ses propres fichiers. Ils peuvent aussi vous aider à réutiliser des
fragments de contenu dans votre application.

Les elements se trouvent dans le dossier ``src/Template/Element/``, et ont une
extension .ctp. Ils sont affichés en utilisant la méthode element de la vue::

    echo $this->element('helpbox');

Passer des Variables à l'intérieur d'un Element
-----------------------------------------------

Vous pouvez passer des données dans un element grâce au deuxième argument::

    echo $this->element('helpbox', [
        "helptext" => "Oh, this text is very helpful."
    ]);

Dans le fichier element, toutes les variables passés sont disponibles comme
des membres du paramètre du tableau (de la même manière que
:php:meth:`Controller::set()` fonctionne dans le controller avec les fichiers
de template). Dans l'exemple ci-dessus, le fichier
``src/Template/Element/helpbox.ctp`` peut utiliser la variable ``$helptext``::

    // A l'intérieur de src/Template/Element/helpbox.ctp
    echo $helptext; //outputs "Oh, this text is very helpful."

La méthode :php:meth:`View::element()` supporte aussi les options pour
l'element. Les options supportées sont 'cache' et 'callbacks'. Un exemple::

    echo $this->element('helpbox', [
            "helptext" => "Ceci est passé à l'element comme $helptext",
            "foobar" => "Ceci est passé à l'element via $foobar",
        ],
        [
            // utilise la configuration de cache "long_view"
            "cache" => "long_view",
            // défini à true pour avoir before/afterRender appelé pour l'element
            "callbacks" => true
        ]
    );

La mise en cache d'element est facilitée par la classe :php:class:`Cache`. Vous
pouvez configurer les elements devant être stockés dans toute configuration de
Cache que vous avez défini. Cela vous donne une grande flexibilité pour
choisir où et combien de temps les elements sont stockés. Pour mettre en cache
les différentes versions du même element dans une application,
fournissez une valeur unique de la clé cache en utilisant le format suivant::

    $this->element('helpbox', [], [
            "cache" => ['config' => 'short', 'key' => 'unique value']
        ]
    );

Vous pouvez tirer profit des elements en utilisant ``requestAction()``. La
fonction ``requestAction()`` récupère les variables de vues à partir
d'une action d'un controller et les retourne en tableau. Cela permet à vos
elements de fonctionner dans un style MVC pur. Créez une action du controller
qui prépare les variables de la vue pour vos elements, ensuite appelez
``requestAction()`` depuis l'intérieur du deuxième paramètre de ``element()``
pour alimenter en variables de vues l'element depuis votre controller.

Pour ce faire, ajoutez quelque chose comme ce qui suit dans votre controller,
en reprenant l'exemple du Post::

    class PostsController extends AppController
    {
        // ...
        public function index()
        {
            $posts = $this->paginate();
            if ($this->request->is('requested')) {
                return $posts;
            } else {
                $this->set('posts', $posts);
            }
        }
    }

Et ensuite dans l'element, nous pouvons accéder au model des posts paginés.
Pour obtenir les cinq derniers posts dans une liste ordonnée, nous ferions
ce qui suit:

.. code-block:: php

    <h2>Latest Posts</h2>
    <?php $posts = $this->requestAction('posts/index?sort=created&direction=asc&limit=5'); ?>
    <ol>
    <?php foreach ($posts as $post): ?>
          <li><?= $post['Post']['title'] ?></li>
    <?php endforeach; ?>
    </ol>

Mise en cache des Elements
--------------------------

Vous pouvez tirer profit de la mise en cache de vue de CakePHP si vous
fournissez un paramètre cache. Si défini à ``true``, cela va mettre en cache
l'element dans la configuration 'default' de Cache. Sinon, vous pouvez définir
la configuration de cache devant être utilisée. Regardez
:doc:`/core-libraries/caching` pour plus d'informations sur la façon de
configurer ``Cache``. Un exemple simple de mise en cache d'un element
serait par exemple::

    echo $this->element('helpbox', [], ['cache' => true]);

Si vous rendez le même element plus d'une fois dans une vue et que vous avez
activé la mise en cache, assurez-vous de définir le paramètre 'key' avec
un nom différent à chaque fois. Cela évitera que chaque appel successif
n'écrase le résultat de la mise en cache du précédent appel de element().
Par exemple::

    echo $this->element(
        'helpbox',
        ['var' => $var],
        ['cache' => ['key' => 'first_use', 'config' => 'view_long']]
    );

    echo $this->element(
        'helpbox',
        ['var' => $differenVar],
        ['cache' => ['key' => 'second_use', 'config' => 'view_long']]
    );

Ce qui est au-dessus va s'enquérir que les deux résultats d'element sont
mis en cache séparément. Si vous voulez que tous les elements mis en cache
utilisent la même configuration du cache, vous pouvez sauvegarder quelques
répétitions, en configurant ``View::$elementCache`` dans la
configuration de Cache que vous souhaitez utiliser. CakePHP va utiliser cette
configuration, quand aucune n'est donnée.

Requêter les Elements à partir d'un Plugin
------------------------------------------

Si vous utilisez un plugin et souhaitez utiliser les elements à partir de
l'intérieur d'un plugin, utilisez juste la :term:`syntaxe de plugin`
habituelle. Si la vue est rendue pour un controller/action d'un plugin, le nom
du plugin va automatiquement être préfixé pour tous les elements utilisés, à
moins qu'un autre nom de plugin ne soit présent.
Si l'element n'existe pas dans le plugin, il ira voir dans le dossier principal
APP. ::

    echo $this->element('Contacts.helpbox');

Si votre vue fait partie d'un plugin, vous pouvez ne pas mettre le nom du
plugin. Par exemple, si vous êtes dans le ``ContactsController`` du plugin
Contacts::

    echo $this->element('helpbox');
    // et
    echo $this->element('Contacts.helpbox');

Sont équivalents et résulteront au même element rendu.

Pour les elements dans le sous-dossier d'un plugin
(e.g., ``plugins/Contacts/sidebar/helpbox.ctp``), utilisez ce qui suit::

    echo $this->element('Contacts.sidebar/helpbox');

Mettre en Cache des Sections de votre View
------------------------------------------

.. php:method:: cache(callable $block, array $options = [])

Parfois, générer une section de l'affichage de votre view peut être couteux
à cause du rendu des :doc:`/views/cells` ou du fait d'opérations de helper
couteuses. Pour que votre application s'exécute plus rapidement, CakePHP fournit
un moyen de mettre en cache des sections de view::

    // En supposant l'existence des variables locales
    echo $this->cache(function () use ($user, $article) {
        echo $this->cell('UserProfile', [$user]);
        echo $this->cell('ArticleFull', [$article]);
    }, ['key' => 'my_view_key']);

Par défaut, le contenu de la view ira dans la config de cache
``View::$elementCache``, mais vous pouvez utiliser l'option ``config`` pour
changer ceci.

Créer vos propres Classes de View
=================================

Vous avez peut-être besoin de créer vos propres classes de vue pour activer des
nouveaux types de données de vue, ou ajouter de la logique supplémentaire
de rendu de vue personnalisée. Comme la plupart des components de CakePHP, les
classes de vue ont quelques conventions:

* Les fichiers de classe de View doivent être mis dans ``src/View``. Par
  exemple ``src/View/PdfView.php``.
* Les classes de View doivent être suffixées avec ``View``. Par exemple
  ``PdfView``.
* Quand vous référencez les noms de classe de vue, vous devez omettre le
  suffixe ``View``. Par exemple ``$this->viewClass = 'Pdf';``.

Vous voudrez aussi étendre ``View`` pour vous assurer que les choses
fonctionnent correctement::

    // dans src/View/PdfView.php

    App::uses('View', 'View');
    class PdfView extends View
    {
        public function render($view = null, $layout = null)
        {
            // logique personnalisée ici.
        }
    }

Remplacer la méthode render vous laisse le contrôle total sur la façon dont
votre contenu est rendu.

En savoir plus sur les vues
===========================

.. toctree::
    :maxdepth: 1

    views/cells
    views/themes
    views/json-and-xml-views
    views/helpers


.. meta::
    :title lang=fr: Views (Vues)
    :keywords lang=fr: logique de vue,fichier csv,éléments de réponse,éléments de code,extension par défaut,json,objet flash,remote application,twig,sous-classe,ajax,répondre,soap,fonctionnalité,cakephp,fréquentation,xml,mvc
