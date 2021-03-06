Navigation panel
================

Context
-------

The navigation panel containing the navigation menu is fully configurable by adding specific services
to the application. Open Orchestra natively provides different kind of strategies for this panel that
are reusable for you own entries.

.. image:: ../../images/navigation_panel.png

Usage
-----

Create the strategy
~~~~~~~~~~~~~~~~~~~

Each panel entry is a service implementing the interface
``OpenOrchestra\Backoffice\NavigationPanel\NavigationPanelInterface``. Different strategies are already
available in the folder ``OpenOrchestra\Backoffice\NavigationPanel\Strategies`` and can be reused for
most common needs in panel services.

For the service to be recognised as a menu entry, it should be tagged with
``open_orchestra_backoffice.navigation_panel.strategy``:

.. code-block:: yaml

    tags:
        - { name: open_orchestra_backoffice.navigation_panel.strategy }

Display of the menu
~~~~~~~~~~~~~~~~~~~

The display of the menu item is managed by the method ``NavigationPanelInterface::show()`` which should
return an HTML string.

The menu link must have the following attributes :

* href : an anchor defining the route for Backbone.js, it will be displayed in the browser's address bar
* id : merge of the "nav-" string and the name of the strategy as returned by
  ``NavigationPanelInterface::getName()``, for instance "nav-user"

It can also have data-* attributes that can help you to define custom properties to be used in a Backbone route,
for example :

* data-url : API url to be called to retrieve the data to be displayed
* data-translated-header : the comma-separated list of column labels in dataTable view (see `Entity list`_)
* data-visible-elements : the comma-separated list of columns, visible by default and identified by id, in dataTable view (see `Entity list`_)
* data-displayed-elements : the comma-separated list of all columns identified by id in dataTable view (see `Entity list`_)
* data-input-header: the comma-separated list of search field of columns (see `Entity list`_)

Define Backbone route
~~~~~~~~~~~~~~~~~~~~~

When clicking on the menu element, the Backbone route matching with href attribute value will be executed.
To learn more about the way to define a new Backbone route, see `Backbone routing in Open Orchestra`_.

Specifics
---------

Order the menus
~~~~~~~~~~~~~~~

It is possible to modify the order of the items in the panel by changing the return value of the
``NavigationPanelInterface::getWeight()`` method. The heaviest elements are displayed below the other ones.

The ``NavigationPanelInterface::getParent()`` method allows an item to have a parent item in order to define
a hierarchy of elements. The root of the panel is an ``administration`` node so all top level items should
have it as a parent.

Access restriction
~~~~~~~~~~~~~~~~~~

Restricting access to a menu element is done by defining a specific role the user should possess inside
``NavigationPanelInterface::getRole()``.

To be allowed to configure the role in the Back Office ( by adding it to a `group`_ ), you will need to add the
role to the ``RoleCollector`` class.

You will need to create a `compiler pass`_ to add the role to the ``RoleCollector`` definition in the ``kernel``.
The ``BackofficeBundle`` already provides the ``AbstractRoleCompilerPass`` which will simplify the process.

Let's say that in the ``FooBundle`` you would like to add the ``ROLE_BAR``.

First create the ``RoleCompilerPass`` in the ``DependencyInjection\Compiler`` folder of your bundle :

.. code-block:: php

    <?php

    namespace OpenOrchestra\FooBundle\DependencyInjection\Compiler;

    use OpenOrchestra\BackofficeBundle\DependencyInjection\Compiler\AbstractRoleCompilerPass;
    use Symfony\Component\DependencyInjection\ContainerBuilder;

    /**
     * Class RoleCompilerPass
     */
    class RoleCompilerPass extends AbstractRoleCompilerPass
    {
        /**
         * You can modify the container here before it is dumped to PHP code.
         *
         * @param ContainerBuilder $container
         *
         * @api
         */
        public function process(ContainerBuilder $container)
        {
            $this->addRoles($container, array(
                'ROLE_BAR',
            ));
        }
    }

Then declare the compiler pass in the bundle creation file ``OpenOrchestraFooBundle`` :

.. code-block:: php

    <?php

    namespace OpenOrchestra\FooBundle;

    use OpenOrchestra\FooBundle\DependencyInjection\Compiler\RoleCompilerPass;
    use Symfony\Component\DependencyInjection\ContainerBuilder;
    use Symfony\Component\HttpKernel\Bundle\Bundle;

    /**
     * Class OpenOrchestraFooBundle
     */
    class OpenOrchestraFooBundle extends Bundle
    {
        /**
         * @param ContainerBuilder $container
         */
        public function build(ContainerBuilder $container)
        {
            parent::build($container);
            $container->addCompilerPass(new RoleCompilerPass());
        }
    }

After clearing the cache, you will see the role ``ROLE_BAR`` displayed in the role list in the ``Group``
modification form.

Finally, you can add some translation on the role. To separate the role translations from the rest of the
application, we use the ``role`` domain. This way, you will have to add the translation in the
``role.en.yml`` file (for the ``en`` locale).

.. _`group`: /en/user_guide/user.rst
.. _`compiler pass`: http://symfony.com/doc/current/cookbook/service_container/compiler_passes.html
.. _`Backbone routing in Open Orchestra`: /en/developer_guide/backbone_routing.rst
.. _`Entity list`: /en/developer_guide/entity_list_ajax_pagination.rst