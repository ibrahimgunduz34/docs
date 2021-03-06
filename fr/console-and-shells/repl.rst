Console Interactive (REPL)
##########################

Le squelette de l'app CakePHP app intègre un REPL(Read Eval Print Loop
= Lire Evaluer Afficher Boucler) qui facilite l'exploration de CakePHP et
de votre application avec une console interactive. Vous pouvez commencer la
console interactive en utilisant::

    $ bin/cake console

Cela va démarrer votre application et lancer une console interactive. A ce
niveau-là, vous pouvez intéragir avec le code de votre application et exécuter
des requêtes en utilisant les models de votre application::

    $ bin/cake console

    Welcome to CakePHP v3.0.0 Console
    ---------------------------------------------------------------
    App : App
    Path: /Users/mark/projects/cakephp-app/src/
    ---------------------------------------------------------------
    [1] app > $articles = Cake\ORM\TableRegistry::get('Articles');
    // object(Cake\ORM\Table)(
    //
    // )
    [2] app > $articles->find();

Puisque votre application a été démarrée, vous pouvez aussi tester le routing
en utilisant le REPL::

    [1] app > Cake\Routing\Router::parse('/articles/view/1');
    // [
    //   'controller' => 'Articles',
    //   'action' => 'view',
    //   'pass' => [
    //     0 => '1'
    //   ],
    //   'plugin' => NULL
    // ]

Vous pouvez aussi tester la génération d'URL::

    [1] app > Cake\Routing\Router::url(['controller' => 'Articles', 'action' => 'edit', 99]);
    // '/articles/edit/99'

Pour quitter le REPL, vous pouver utiliser ``CTRL-D``.

.. warning::

    Le REPL ne fonctionne pas correctement sur les systèmes windows en raison
    de problèmes avec la readline et les extensions POSIX.
