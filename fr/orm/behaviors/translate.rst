Translate
#########

.. php:namespace:: Cake\ORM\Behavior

.. php:class:: TranslateBehavior

Le behavior Translate vous permet de créer et de récupérer les copies traduits
de vos entities en plusieurs langues. Il le fait en utilisant une table
``i18n`` séparée où il stocke la traduction pour chacun des champs de tout
objet Table donné auquel il est lié.

.. warning::
    TranslateBehavior ne supporte pas les clés primaires composite pour
    l'intant.

Un Rapide Aperçu
================

Après avoir créé la table ``i18n`` dans votre base de données, attachez le
behavior à l'objet Table que vous souhaitez rendre traduisible::

    class ArticlesTable extends Table
    {
    
        public function initialize(array $config)
        {
            $this->addBehavior('Translate', ['fields' => ['title']]);
        }
    }

Maintenant, sélectionnez une langue à utiliser pour récupérer les entities::

    I18n::locale('spa');
    $articles = TableRegistry::get('Articles');

Ensuite, récupérez une entity existante::

    $article = $articles->get(12);
    echo $article->title; // Affiche 'A title', pas encore traduit

Ensuite, traduisez votre entity::

    $article->title = 'Un Artículo';
    $articles->save($article);

Vous pouvez maintenant essayer de récupérer à nouveau votre entity::

    $article = $articles->get(12);
    echo $article->title; // Affiche 'Un Artículo', ouah facile!

Travailler avec plusieurs traductions peut être fait facilement en utilisant un
trait spécial dans votre classe Entity::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Maintenant, vous pouvez trouver toutes les traductions pour une entity unique::

    $article = $articles->find('translations')->first();
    echo $article->translation('spa')->title; // 'Un Artículo'

    echo $article->translation('eng')->title; // 'An Article';

Il est également facile de sauvegarder plusieurs traductions en une fois::

    $article->translation('spa')->title = 'Otro Título';
    $article->translation('fre')->title = 'Un autre Titre';
    $articles->save($articles);

Oui, aussi facilement. Si vous voulez aller plus en profondeur sur la façon
dont il fonctionne ou pour affiner le behavior à vos besoins, continuez de
lire le reste de ce chapitre.

Initialiser la Table i18n de la Base de Données
===============================================

Afin d'utiliser le behavior, vous avez besoin de créer une table ``i18n`` avec
le bon schéma. Habituellement, la seule façon de charger la table ``i18n`` est
en lançant manuellement le script SQL suivant dans votre base de données:

.. code-block:: sql

    CREATE TABLE i18n (
        id int NOT NULL auto_increment,
        locale varchar(6) NOT NULL,
        model varchar(255) NOT NULL,
        foreign_key int(10) NOT NULL,
        field varchar(255) NOT NULL,
        content text,
        PRIMARY KEY	(id),
        UNIQUE INDEX I18N_LOCALE_FIELD(locale, model, foreign_key, field),
        INDEX I18N_FIELD(model, foreign_key, field)
    );


Attacher le Behavior Translate à Vos Tables
===========================================

Attacher le behavior peut être fait dans la méthode ``initialize`` dans votre
classe Table::

    class Articles extends Table
    {
    
        public function initialize(array $config)
        {
            $this->addBehavior('Translate', ['fields' => ['title', 'body']]);
        }
    }

La première chose à noter est que vous devez passer la clé ``fields`` dans le
tableau de configuration. La liste des champs est souhaitée pour dire au
behavior les colonnes qui seront capable de stocker les traductions.

Utiliser une Table de Traductions Séparée
-----------------------------------------

Si vous souhaitez utiliser une table autre que ``i18n`` pour la traduction
d'un dépôt particulier, vous pouvez le spécifier dans la configuration du
behavior. C'est commun quand vous avez plusieurs tables à traduire et que vous
souhaitez une séparation propre des données qui est stocké pour chaque table
différente::


    class Articles extends Table
    {
    
        public function initialize(array $config)
        {
            $this->addBehavior('Translate', [
                'fields' => ['title', 'body'],
                'translationTable' => 'ArticlesI18n'
            ]);
        }
    }

Vous avez besoin de vous assurer que toute table personnalisée que vous utilisez
a les colonnes ``field``, ``foreign_key``, ``locale`` et ``model``.

Lire du Contenu Traduit
=======================

Comme montré ci-dessus, vous pouvez utiliser la méthode ``locale`` pour choisir
la traduction active pour les entities qui sont chargées::
translation for entities that are loaded::

    I18n::locale('spa');
    $articles = TableRegistry::get('Articles');

    // Toutes les entities dans les résultats vont contenir la traduction espagnol
    $results = $articles->find()->all();

Cette méthode fonctionne avec tout finder dans vos tables. Par exemple, vous
pouvez utiliser TranslateBehavior avec ``find('list')``::

    I18n::locale('spa');
    $data = $articles->find('list')->toArray();

    // Data va contenir
    [1 => 'Mi primer artículo', 2 => 'El segundo artículo', 15 => 'Otro articulo' ...]

Récupérer Toutes les Traductions Pour Une Entity
------------------------------------------------

Lorsque vous construisez des interfaces pour la mise à jour de contenu traduite,
il est souvent utile de montrer une ou plusieurs traduction(s) au même moment.
Vous pouvez utiliser le finder ``translations`` pour ceci::

    // Récupère le premier article avec toutes les traductions correspondantes
    $article = $articles->find('translations')->first();

Dans l'exemple ci-dessus, vous obtiendrez une liste d'entities en retour qui
a une propriété ``_translations`` définie. Cette propriété va contenir une liste
d'entities de données traduites. Par exemple, les propriétés suivantes seront
accessibles::

    // Affiche 'eng'
    echo $article->_translations['eng']->locale;

    // Affiche 'title'
    echo $article->_translations['eng']->field;

    // Affiche 'My awesome post!'
    echo $article->_translations['eng']->body;

Une façon plus élégante pour gérer les données est d'ajouter un trait pour la
classe entity qui est utilisé pour votre table::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Ce trait contient une méthode unique appelée ``translation``, ce qui vous laisse
accéder ou créer des entities de nouvel traduction à la volée::

    // Affiche 'title'
    echo $article->translation('eng')->title;

    // Ajoute une nouvelle données de traduction de l'entity à l'article
    $article->translation('deu')->title = 'Wunderbar';

Limiter les Traductions à Récupérer
-----------------------------------

Vous pouvez limiter les langues que vous récupérez à partir de la base de
données pour un ensemble particulier d'enregistrements::

    $results = $articles->find('translations', ['locales' => ['eng', 'spa']]);
    $article = $results->first();
    $spanishTranslation = $article->translation('spa');
    $englishTranslation = $article->translation('eng');

Récupérer Toutes les Traductions pour des Associations
------------------------------------------------------

Il est aussi possible de trouver des traductions pour toute association dans une
opération de find unique::

    $article = $articles->find('translations')->contain([
        'Categories' => function ($query) {
            return $query->find('translations');
        }
    ])->first();

    // Affiche 'Programación'
    echo $article->categories[0]->translation('spa')->name;

Ceci implique que ``Categories`` a le TranslateBehavior attaché à celui-ci. Il
utilise simplement la fonction de construction de requête pour la clause
``contain`` d'utiliser les ``translations`` du finder personnalisé dans
l'association.

Récupérer une langue sans utiliser I18n::locale
-----------------------------------------------

Appeler ``I18n::locale('spa');`` change la locale par défaut pour tous les finds
traduits, il peut y avoir des fois où vous souhaitez récupérer du contenu
traduit sans modification de l'état de l'application. Pour ces scenarii,
utilisez la méthode ``locale`` du behavior::

    I18n::locale('eng'); // reset for illustration
    $articles = TableRegistry::get('Articles');
    $articles->locale('spa'); // specific locale

    $article = $articles->get(12);
    echo $article->title; // Echoes 'Un Artículo', yay piece of cake!

Notez que ceci va seulement changer la locale de la table Articles, cela ne
changera pas la langue des données associées. Pour utiliser cette technique
pour changer les données associées, il est nécessaire d'appeler la locale
pour chaque table par exemple::

    I18n::locale('eng'); // reset for illustration
    $articles = TableRegistry::get('Articles');
    $articles->locale('spa');
    $articles->categories->locale('spa');

    $data = $articles->find('all', ['contain' => ['Categories']]);

Cet exemple suppose que ``Categories`` a le TranslateBehavior attaché.

Sauvegarder dans une Autre Langue
=================================

La philosophie derrière le TranslateBehavior est que vous avez une entity
représentant la langue par défaut, et plusieurs traductions qui peuvent
surcharger certains champs dans de tels entities. Garder ceci à l'esprit, vous
pouvez sauvegarder de façon intuitive les traductions pour une entity donnée.
Par exemple, étant donné la configuration suivante::

    class Articles extends Table
    {
        public function initialize(array $config)
        {
            $this->addBehavior('Translate', ['fields' => ['title', 'body']]);
        }
    }

    class Article extends Entity
    {
        use TranslateTrait;
    }

    $articles = TableRegistry::get('Articles');
    $article = new Article([
        'title' => 'My First Article',
        'body' => 'This is the content',
        'footnote' => 'Some afterwords'
    ]);

    $articles->save($article);

Donc, après avoir sauvegardé votre premier article, vous pouvez maintenant
sauvegarder une traduction pour celui-ci, il y a quelques façons de le faire. La
première est de configurer la langue directement dans une entity::

    $article->_locale = 'spa';
    $article->title = 'Mi primer Artículo';

    $articles->save($article);

Après que l'entity a été sauvegardé, le champ traduit va aussi être persistent,
une chose à noter est que les valeurs à partir de la langue par défaut qui
étaient surchargées seront préservées::

    // Affiche 'This is the content'
    echo $article->body;

    // Affiche 'Mi primer Artículo'
    echo $article->title;

Une fois que vous surchargez la valeur, la traduction pour ce champ sera
sauvegardée et récupérée comme d'habitude::

    $article->body = 'El contendio';
    $articles->save($article);

La deuxième manière de l'utiliser pour sauvegarder les entities dans une autre
langue est de définir la langue par défaut directement à la table::

    I18n::locale('spa');
    $article->title = 'Mi Primer Artículo';
    $articles->save($article);

Configurer la langue directement dans la table est utile quand vous avez besoin
à la fois de récupérer et de sauvegarder les entities pour la même langue
ou quand vous avez besoin de sauvegarder plusieurs entities en une fois.

Sauvegarder Plusieurs Traductions
=================================

C'est un pré-requis habituel d'être capable d'ajouter ou de modifier plusieurs
traductions à l'enregistrement de la base de données au même moment. Ceci peut
être facilement fait en utilisant ``TranslateTrait``::

    use Cake\ORM\Behavior\Translate\TranslateTrait;
    use Cake\ORM\Entity;

    class Article extends Entity
    {
        use TranslateTrait;
    }

Maintenant vous pouvez ajouter les translations avant de les sauvegarder::

    $translations = [
        'fra' => ['title' => "Un article"],
        'spa' => ['title' => 'Un artículo']
    ];

    foreach ($translations as $lang => $data) {
        $article->translation($lang)->set($data, ['guard' => false]);
    }

    $articles->save($article);
