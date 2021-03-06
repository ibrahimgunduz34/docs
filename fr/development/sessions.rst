Sessions
########

CakePHP fournit des fonctionnalités en plus et une suite d'utilitaires au-dessus
de l'extension native ``session`` de PHP. Les Sessions vous permettent
d'identifier les utilisateurs uniques pendant leurs requêtes et de stocker les
données persistantes pour les utilisateurs spécifiques. Au contraire des
Cookies, les données de session ne sont pas disponibles du coté client.
L'utilisation de ``$_SESSION`` est généralement à éviter dans CakePHP, et à
la place l'utilisation des classes de Session est préférable.

.. _session-configuration:

Session Configuration
=====================

La configuration de Session est stockée dans ``Configure`` dans la clé de top
niveau ``Session``, et un certain nombre d'options sont disponibles:

* ``Session.timeout`` - Le nombre de *minutes* avant que le gestionnaire de
  session de CakePHP ne fasse expirer la session.

* ``Session.defaults`` - Vous permet d'utiliser les configurations de session
  intégrées par défaut comme une base pour votre configuration de session.
  Regardez ci-dessous les paramètres intégrés par défaut

* ``Session.handler`` - Vous permet de définir un gestionnaire de session
  personnalisé. La base de données du cœur et les gestionnaires de cache
  de session utilisent celui-ci. Regardez ci-dessous pour des informations
  supplémentaires sur les gestionnaires de Session.

* ``Session.ini`` - Vous permet de définir les configurations ini de session
  supplémentaire pour votre config. Ceci combiné avec ``Session.handler``
  remplace les fonctionnalités de gestionnaire de session personnalisé
  des versions précédentes.

CakePHP met par défaut la configuration de ``session.cookie_secure`` à ``true``,
quand votre application est sur un protocole SSL. Si votre application sert
à partir des deux protocoles SSL et non-SSL, alors vous aurez peut-être
des problèmes avec les sessions étant perdues. Si vous avez besoin d'accéder
à la session sur les deux domaines SSL et non-SSL, vous aurez envie de
désactiver cela::

    Configure::write('Session', [
        'defaults' => 'php',
        'ini' => [
            'session.cookie_secure' => false
        ]
    ]);

Le chemin du cookie de session est par défaut le chemin de base de
l'application. Pour changer ceci, vous pouvez utiliser la valeur ini
``session.cookie_path``. Par exemple, si vous voulez que votre session soit
sauvegardée pour tous les sous-domaines, vous pouvez faire::

    Configure::write('Session', [
        'defaults' => 'php',
        'ini' => [
            'session.cookie_path' => '/',
            'session.cookie_domain' => '.yourdomain.com'
        ]
    ]);

Par défaut PHP définit le cookie de session pour qu'il expire dès que le
navigateur est fermé, quelque soit la valeur ``Session.timeout`` configurée.
Le timeout du cookie est contrôlé par la valeur ini ``session.cookie_lifetime``
et peut être configuré en utilisant::

    Configure::write('Session', [
        'defaults' => 'php',
        'ini' => [
            // Rend le cookie non valide après 30 minutes s'il n'y
            // a aucune visite d'aucune page sur le site.
            'session.cookie_lifetime' => 1800
        ]
    ]);

La différence entre les valeurs ``Session.timeout`` et
``session.cookie_lifetime`` est que la deuxième repose sur le fait que le
client dit la vérité sur le cookie. Si vous devez vérifier plus strictement
le timeout, sans que cela ne repose sur ce que dit le client, vous devez
utiliser ``Session.timeout``.

Merci de noter que ``Session.timeout`` correspond au temps total d'inactivité
d'un utilisateur (par ex, le temps sans visite d'aucune page où la session
est utilisée), et ne limite pas le nombre total de minutes pendant lesquelles
un utilisateur peut rester sur le site.

Gestionnaires de Session intégrés & Configuration
=================================================

CakePHP est fourni avec plusieurs configurations de session intégrées. Vous
pouvez soit utiliser celles-ci comme base pour votre configuration de
session, soit vous pouvez créer une solution complètement personnalisée.
Pour utiliser les valeurs par défaut, définissez simplement la clé
'defaults' avec le nom par défaut que vous voulez utiliser. Vous pouvez
ensuite surcharger toute sous-configuration en la déclarant dans votre config
Session::

    Configure::write('Session', [
        'defaults' => 'php'
    ]);

Ce qui est au-dessus va utiliser la configuration de session intégrée dans
'php'. Vous pourriez augmenter tout ou partie de celle-ci en faisant
ce qui suit::

    Configure::write('Session', [
        'defaults' => 'php',
        'cookie' => 'my_app',
        'timeout' => 4320 // 3 days
    ]);

Ce qui est au-dessus surcharge le timeout et le nom du cookie pour la
configuration de session 'php'. Les configurations intégrées sont:

* ``php`` - Sauvegarde les sessions avec les configurations standard dans
  votre fichier php.ini.
* ``cake`` - Sauvegarde les sessions en tant que fichiers à l'intérieur de
  ``app/tmp/sessions``. Ceci est une bonne option quand les hôtes ne
  vous autorisent pas à écrire en dehors de votre propre dir home.
* ``database`` - Utilise les sessions de base de données intégrées.
  Regardez ci-dessous pour plus d'informations.
* ``cache`` - Utilise les sessions de cache intégrées. Regardez
  ci-dessous pour plus d'informations.

Gestionnaires de Session
------------------------

Les gestionnaires peuvent aussi être définis dans le tableau de config de
session. En définissant la clé de config 'handler.engine', vous pouvez nommer
le nom de la classe, ou fournir une instance de gestionnaire. La classe/objet
doit implémenter le ``SessionHandlerInterface`` natif de PHP. Implémenter
cette interface va permettre de faire le lien automatiquement de
``Session`` vers les méthodes du gestionnaire. Le Cache du cœur et les
gestionnaires de session de la Base de Données utilisent tous les deux cette
méthode pour sauvegarder les sessions. De plus, les configurations pour le
gestionnaire doivent être placées dans le tableau du gestionnaire. Vous
pouvez ensuite lire ces valeurs à partir de votre gestionnaire::

    'Session' => [
        'handler' => [
            'engine' => 'Database',
            'model' => 'CustomSessions'
        ]
    ]

Ce qui est au-dessus montre comment vous pouvez configurer le gestionnaire
de session de la Base de Données avec un model de l'application. Lors de
l'utilisation de noms de classe comme handler.engine, CakePHP va s'attendre
à trouver votre classe dans le namespace ``Network\\Session``. Par exemple,
si vous aviez une classe ``AppSessionHandler``, le fichier doit être
``src/Network/Session/AppSessionHandler.php``, et le nom de classe doit être
``App\\Network\\Session\\AppSessionHandler``. Vous pouvez aussi utiliser les
gestionnaires de session à partir des plugins. En configurant le moteur
avec ``MyPlugin.PluginSessionHandler``.

Les Sessions de la Base de Données
----------------------------------

Les changements dans la configuration de session changent la façon dont vous
définissez les sessions de base de données.
La plupart du temps, vous aurez seulement besoin de définir
``Session.handler.model`` dans votre configuration ainsi que
choisir la base de données par défaut::

    Configure::write('Session', [
        'defaults' => 'database',
        'handler' => [
            'model' => 'CustomSessions'
        ]
    ]);

Ce qui est au-dessus va dire à Session d'utiliser le 'database' intégré
par défaut, et spécifier qu'un model appelé ``CustomSession`` sera celui
délégué pour la sauvegarde d'information de session dans la base de données.

Si vous n'avez pas besoin d'un gestionnaire de session complètement
personnalisable, mais que vous avez tout de même besoin de stockage de session
en base de données, vous pouvez simplifier le code du dessus par
celui-ci::

    Configure::write('Session', [
        'defaults' => 'database'
    ]);

Cette configuration nécessitera qu'une table de base de données soit ajoutée
avec au moins ces champs::

    CREATE TABLE `sessions` (
      `id` varchar(255) NOT NULL DEFAULT '',
      `data` text,
      `expires` int(11) DEFAULT NULL,
      PRIMARY KEY (`id`)
    );

Vous pouvez trouver une copie du schéma pour la table de sessions dans le
squelette d'application.

Les Sessions de Cache
---------------------

La classe Cache peut aussi être utilisée pour stocker les sessions. Cela vous
permet de stocker les sessions dans un cache comme APC, memcache, ou Xcache.
Il y a quelques bémols dans l'utilisation des sessions en cache, puisque
si vous vider le cache, les sessions vont commencer à expirer
puisque les enregistrements sont évincés.

Pour utiliser les sessions basées sur le Cache, vous pouvez configurer votre
config Session comme ceci ::

    Configure::write('Session', [
        'defaults' => 'cache',
        'handler' => [
            'config' => 'session'
        ]
    ]);


Cela va configurer Session pour utiliser la classe ``CacheSession``
déléguée pour sauvegarder les sessions. Vous pouvez utiliser la clé 'config'
qui va mettre en cache la configuration à utiliser. La configuration par
défaut de la mise en cache est ``'default'``.

Configurer les Directives ini
=============================

Celui intégré par défaut tente de fournir une base commune pour la
configuration de session. Vous aurez aussi besoin d'ajuster les flags ini
spécifiques. CakePHP donne la capacité de personnaliser les configurations
ini pour les deux configurations par défaut, ainsi que celles personnalisées.
La clé ``ini`` dans les configurations de session vous permet de spécifier les
valeurs de configuration individuelles. Par exemple vous pouvez l'utiliser
pour contrôler les configurations comme ``session.gc_divisor``::

    Configure::write('Session', [
        'defaults' => 'php',
        'ini' => [
            'session.cookie_name' => 'MyCookie',
            'session.cookie_lifetime' => 1800, // Valide pour 30 minutes
            'session.gc_divisor' => 1000,
            'session.cookie_httponly' => true
        ]
    ]);


Créer un Gestionnaire de Session Personnalisé
=============================================

Créer un gestionnaire de session personnalisé est simple dans CakePHP. Dans cet
exemple, nous allons créer un gestionnaire de session qui stocke les sessions
à la fois dans le Cache (apc) et la base de données. Cela nous donne le
meilleur du IO rapide de apc, sans avoir à se soucier des sessions s'évaporant
quand le cache se remplit.

D'abord, nous aurons besoin de créer notre classe personnalisée et de la
mettre dans ``src/Network/Session/ComboSession.php``. La classe
devrait ressembler à::

    namespace App\Network\Session;

    use Cake\Cache\Cache;
    use Cake\Core\Configure;
    use Cake\Network\Session\DatabaseSession;

    class ComboSession extends DatabaseSession
    {
        public $cacheKey;

        public function __construct()
        {
            $this->cacheKey = Configure::read('Session.handler.cache');
            parent::__construct();
        }

        // Lire des données de session.
        public function read($id)
        {
            $result = Cache::read($id, $this->cacheKey);
            if ($result) {
                return $result;
            }
            return parent::read($id);
        }

        // Ecrire des données dans session
        public function write($id, $data)
        {
            Cache::write($id, $data, $this->cacheKey);
            return parent::write($id, $data);
        }

        // Détruire une session.
        public function destroy($id)
        {
            Cache::delete($id, $this->cacheKey);
            return parent::destroy($id);
        }

        // Retire des sessions expirées.
        public function gc($expires = null)
        {
            return Cache::gc($this->cacheKey) && parent::gc($expires);
        }
    }

Notre classe étend la classe intégrée ``DatabaseSession`` donc nous ne devons
pas dupliquer toute sa logique et son comportement. Nous entourons chaque
opération avec une opération :php:class:`Cake\\Cache\\Cache`. Cela nous laisse
récupérer les sessions de la mise en cache rapide, et nous évite de nous
inquiéter sur ce qui arrive quand nous remplissons le cache. Utiliser le
gestionnaire de session est aussi facile. Dans votre ``app.php`` imitez le
block de session ressemblant à ce qui suit::

    'Session' => [
        'defaults' => 'database',
        'handler' => [
            'engine' => 'ComboSession',
            'model' => 'Session',
            'cache' => 'apc'
        ]
    ],
    // Assurez-vous d'ajouter une config de cache apc
    'Cache' => [
        'apc' => ['engine' => 'Apc']
    ]

Maintenant notre application va commencer en utilisant notre gestionnaire
de session personnalisé pour la lecture & l'écriture des données de session.

.. php:class:: Session

.. _accessing-session-object:

Accéder à l'Objet Session
=========================

Vous pouvez accéder aux données session à tous les endroits où vous avez accès
à l'objet request. Cela signifie que la session est facilement accessible via::

* Controllers
* Views
* Helpers
* Cells
* Components

En plus de l'objet basique session, vous pouvez aussi utiliser
:php:class:`Cake\\View\\Helper\\SessionHelper` pour interagir avec la session
dans vos views. Un exemple simple de l'utilisation de session serait::

    $name = $this->request->session()->read('User.name');

    // Si vous accédez à la session plusieurs fois,
    // vous voudrez probablement une variable locale.
    $session = $this->request->session();
    $name = $session->read('User.name');

Lire & Ecrire les Données de Session
====================================

.. php:staticmethod:: read($key)

Vous pouvez lire les valeurs de session en utilisant la syntaxe
compatible :php:meth:`Hash::extract()`::

     $session->read('Config.language');

.. php:staticmethod:: write($key, $value)

``$key`` devrait être le chemin séparé de point et ``$value`` sa valeur::

     $session->write('Config.language', 'eng');

.. php:staticmethod:: delete($key)

Quand vous avez besoin de supprimer des données de la session, vous pouvez
utiliser ``delete()``::

     $session->delete('Some.value');

.. php:staticmethod:: consume($key)

Quand vous avez besoin de lire et supprimer des données de la session, vous
pouvez utiliser ``consume()``::

    $session->consume('Some.value');

.. php:method:: check($key)

Si vous souhaitez voir si des données existent dans la session, vous pouvez
utiliser ``check()``::

    if ($session->check('Config.language')) {
        // Config.language existe et n'est pas null.
    }

Détruire la Session
===================

.. php:method:: destroy()

Détruire la session est utile quand les utilisateurs de déconnectent. Pour
détruire une session, utilisez la méthode ``destroy()``::

    $session->destroy();

Détruire une session va retirer toutes les données sur le serveur dans la
session, mais **ne va pas** retirer le cookie de session.

Faire une Rotation des Identificateurs de Session
=================================================

.. php:method:: renew()

Alors que ``AuthComponent`` réactualise automatiquement l'id de session quand
les utilisateurs se connectent et se déconnectent, vous aurez peut-être besoin
de faire une rotation de l'id de session manuellement. Pour ce faire, utilisez
la méthode ``renew()``::

    $session->renew();

Messages Flash
==============

Les messages flash sont des messages courts à afficher aux utilisateurs une
seule fois. Ils sont souvent utilisés pour afficher des messages d'erreur ou
pour confirmer que les actions se font avec succès.

Pour définir et afficher les messages flash, vous devez utiliser
:doc:`/controllers/components/flash` et
:doc:`/views/helpers/flash`

.. meta::
    :title lang=fr: Sessions
    :keywords lang=fr: session defaults,session classes,utility features,session timeout,session ids,persistent data,session key,session cookie,session data,last session,core database,security level,useragent,security reasons,session id,attr,countdown,regeneration,sessions,config
