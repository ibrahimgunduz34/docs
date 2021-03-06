View Cells
##########

View cells are small mini-controllers that can invoke view logic and render out
templates. They provide a light-weight modular replacement to
``requestAction()``. The idea of cells is borrowed from `cells in Ruby
<https://github.com/apotonick/cells>`_, where they fulfill a similar role and purpose.

When to use Cells
=================

Cells are ideal for building reusable page components that require interaction
with models, view logic, and rendering logic. A simple example would be the
cart in an online store, or a data-driven navigation menu in a CMS. Because
cells do not dispatch sub-requests, they sidestep all of the overhead associated
with ``requestAction()``.

Creating a Cell
===============

To create a cell, define a class in ``src/View/Cell`` and a template in
``src/Template/Cell/``. In this example, we'll be making a cell to display the
number of messages in a user's notification inbox. First, create the class file.
Its contents should look like::

    namespace App\View\Cell;

    use Cake\View\Cell;

    class InboxCell extends Cell
    {

        public function display()
        {
        }

    }

Save this file into ``src/View/Cell/InboxCell.php``. As you can see, like other
classes in CakePHP, Cells have a few conventions:

* Cells live in the ``App\View\Cell`` namespace. If you are making a cell in
  a plugin, the namespace would be ``PluginName\View\Cell``.
* Class names should end in Cell.
* Classes should inherit from ``Cake\View\Cell``.

We added an empty ``display()`` method to our cell; this is the conventional
default method when rendering a cell. We'll cover how to use other methods later
in the docs. Now, create the file ``src/Template/Cell/Inbox/display.ctp``. This
will be our template for our new cell.

You can generate this stub code quickly using ``bake``::

    bin/cake bake cell Inbox

Would generate the code we typed out.

Implementing the Cell
---------------------

Assume that we are working on an application that allows users to send messages
to each other. We have a ``Messages`` model, and we want to show the count of
unread messages without having to pollute AppController. This is a perfect use
case for a cell. In the class we just made, add the following::

    namespace App\View\Cell;

    use Cake\View\Cell;

    class InboxCell extends Cell
    {

        public function display()
        {
            $this->loadModel('Messages');
            $unread = $this->Messages->find('unread');
            $this->set('unread_count', $unread->count());
        }

    }

Because Cells use the ``ModelAwareTrait`` and ``ViewVarsTrait``, they behave
very much like a controller would.  We can use the ``loadModel()`` and ``set()``
methods just like we would in a controller. In our template file, add the
following::

    <!-- src/Template/Cell/Inbox/display.ctp -->
    <div class="notification-icon">
        You have <?= $unread_count ?> unread messages.
    </div>

.. note::

    Cell templates have an isolated scope that does not share the same View
    instance as the one used to render template and layout for the current
    controller action or other cells. Hence they are unaware of any helper calls
    made or blocks set in the action's template / layout and vice versa.

Loading Cells
=============

Cells can be loaded from views using the ``cell()`` method and works the same in
both contexts::

    // Load an application cell
    $cell = $this->cell('Inbox');

    // Load a plugin cell
    $cell = $this->cell('Messaging.Inbox');

The above will load the named cell class and execute the ``display()`` method.
You can execute other methods using the following::

    // Run the expanded() method on the Inbox cell
    $cell = $this->cell('Inbox::expanded');

If you need controller logic to decide which cells to load in a request, you can
use the ``CellTrait`` in your controller to enable the ``cell()`` method there::

    namespace App\Controller;

    use App\Controller\AppController;
    use Cake\View\CellTrait;

    class DashboardsController extends AppController
    {
        use CellTrait;

        // More code.
    }

Passing Arguments to a Cell
---------------------------

You will often want to parameterize cell methods to make cells more flexible.
By using the second and third arguments of ``cell()``, you can pass action
parameters and additional options to your cell classes::

    $cell = $this->cell('Inbox::recent', ['since' => '-3 days']);

The above would match the following function signature::

    public function recent($since)
    {
    }

Rendering a Cell
================

Once a cell has been loaded and executed, you'll probably want to render it. The
easiest way to render a cell is to echo it::

    <?= $cell ?>

This will render the template matching the lowercased and underscored version of
our action name, e.g. ``display.ctp``.

Because cells use ``View`` to render templates, you can load additional cells
within a cell template if required.

Rendering Alternate Templates
-----------------------------

By convention cells render templates that match the action they are executing.
If you need to render a different view template, you can specify the template
to use when rendering the cell::

    // Calling render() explicitly
    echo $this->cell('Inbox::recent', ['since' => '-3 days'])->render('messages');

    // Set template before echoing the cell.
    $cell = $this->cell('Inbox');
    $cell->template = 'messages';
    echo $cell;

Caching Cell Output
-------------------

When rendering a cell you may want to cache the rendered output if the contents
don't change often or to help improve performance of your application. You can
define the ``cache`` option when creating a cell to enable & configure caching::

    // Cache using the default config and a generated key
    $cell = $this->cell('Inbox', [], ['cache' => true]);

    // Cache to a specific cache config and a generated key
    $cell = $this->cell('Inbox', [], ['cache' => ['config' => 'cell_cache']]);

    // Specify the key and config to use.
    $cell = $this->cell('Inbox', [], [
        'cache' => ['config' => 'cell_cache', 'key' => 'inbox_' . $user->id]
    ]);

If a key is generated the underscored version of the cell class and template
name will be used.
