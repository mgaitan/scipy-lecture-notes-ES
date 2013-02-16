======
Traits
======

:author: Didrik Pinte

El proyecto Traits le permite añadir de manera simple validación, inicialización, delegación, notificación y una interfaz gráfica de usuario a atributos de objetos Python. En este tutorial exploraremos el conjunto de herramientas de Traits y aprenderemos como reducir dramáticamente la cantidad de código *boilerplate*, para un desarrollo rápido de aplicaciones GUI, comprendiendo las ideas que subyacen en otras partes del Enthought Tool Suite.

Traits y el Enthought Tool Suite son proyectos de código abierto bajo una licencia estilo BSD.

.. topic:: Público destinatario

    Programadores Python de nivel intermedio o avanzado

.. topic:: Requirimientos

    * Python 2.6 or 2.7 (www.python.org)
    * wxPython (http://www.wxpython.org/) o PyQt (http://www.riverbankcomputing.co.uk/software/pyqt/intro)
    * Numpy y Scipy (http://www.scipy.org)
    * Enthought Tool Suite 3.x o superior (http://code.enthought.com/projects)
    * Todo el software requerido es provisto con la instalación de EPD Free (http://www.enthought.com/products/epd.php)


.. contents:: Contenido del tutorial
   :local:
   :depth: 2


Introducción
============

El Enthought Tool Suite posibilita la construcción de sofisticadas *frameworks* de aplicaciones para análisis de datos, graficación 2D y visualización 3D. Estos potentes y reusables componentes están liberados bajo una permisiva licencia estilo BSD.

Los principales paquetes son:

    * Traits - aproximación basada en componentes para la construcción de aplicaciones.
    * Kiva - primitivas 2D incluyendo render basado en ramas,
      transformación afín, fundido de canal alfa, y más
    * Enable - Canvas de dibujado 2D basado en objetos.
    * Chaco - Toolkit de graficación para la construcción de complejos gráficos 2D interactivos.
    * Mayavi - Visualización 3D de datos científicos basado en VTK.
    * Envisage - Framework de plugins de aplicaciones para construir software automatizable y extensible

.. image:: ETS.jpg
    :align: center

En este tutorial nos enfocaremos en Traits.

Ejemplo
=======

A lo largo de este tutorial, usaremos un ejemplo basado en un simple caso
de administración de recursos hídricos. Intentaremos modelar una represa
y un sistema de reservorio. El reservorio y las represas tienen un conjunto
de parámetros:

    * Nombre
    * Capacidad mínima y máxima del reservorio [hm3]
    * Altura y largo de la represa [m]
    * Superficie de la cuenca [km2]
    * Cabezal hidráulico [m]
    * Potencia de turbinas [MW]
    * Caudal mínimo y máximo [m3/s]
    * Eficiencia de las turbinas

El reservorio tiene un comportamiento conocido. Una parte está relacionada
a la energía de producción basada en el caudal de agua. Una simple formula
para obtener una aproximación de la energía eléctrica producida en una
planta hidroeléctrica es :math:`P = \rho hrgk`, donde:

    * :math:`P` es la potencia en watts,
    * :math:`\rho` es la densidad del agua (~1000 kg/m3),
    * :math:`h` es la altura en metros,
    * :math:`r` es el caudal medio en metros cúbicos por segundo,
    * :math:`g` es la aceleración de la gravedad: 9.8 m/s2,
    * :math:`k` es el coeficiente de eficiencia, entre 0 y 1.

La producción de energía anual depende de la provisión agua disponible.
En algunas instalaciones el flujo de agua puede variar en un factor 10:1
a lo largo del año.

La segunda parte del comportamiento es el estado de la cuenca, que depende
de parámetros controlados y no controlados:

The second part of the behaviour is the state of the storage that depends on
controlled and uncontrolled parameters :

    :math:`storage_{t+1} = storage_t + inflows - release - spillage - irrigation`

.. warning::

    The data used in this tutorial are not real and might even not have sense
    in the reality.

What are Traits
===============

A trait is a type definition that can be used for normal Python object attributes, giving the attributes some additional characteristics:

    * Standardization:
        * Initialization
        * Validation
        * Deferral
    * Notification
    * Visualization
    * Documentation

A class can freely mix trait-based attributes with normal Python attributes, or can opt to allow the use of only a fixed or open set of trait attributes within the class. Trait attributes defined by a class are automatically inherited by any subclass derived from the class.

The common way of creating a traits class is by extending from the
**HasTraits** base class and defining class traits :

::

    from traits.api import HasTraits, Str, Float

    class Reservoir(HasTraits):

        name = Str
        max_storage = Float


.. warning:: For Traits 3.x users

    If using Traits 3.x, you need to adapt the namespace of the traits
    packages:

        * traits.api should be enthought.traits.api
        * traitsui.api should be enthought.traits.ui.api

Using a traits class like that is as simple as any other Python class. Note
that the trait value are passed using keyword arguments:

::

    reservoir = Reservoir(name='Lac de Vouglans', max_storage=605)



Initialisation
--------------

All the traits do have a default value that initialise the variables. For
example, the basic python types do have the following trait equivalents:

==============  ====================== ======================
Trait           Python Type            Built-in Default Value
==============  ====================== ======================
Bool            Boolean                False
Complex         Complex number         0+0j
Float           Floating point number  0.0
Int             Plain integer          0
Long            Long integer           0L
Str             String                 ''
Unicode         Unicode                u''
==============  ====================== ======================

A number of other predefined trait type do exist : Array, Enum, Range, Event,
Dict, List, Color, Set, Expression, Code, Callable, Type, Tuple, etc.

Custom default values can be defined in the code:

::

    from traits.api import HasTraits, Str, Float

    class Reservoir(HasTraits):

        name = Str
        max_storage = Float(100)

    reservoir = Reservoir(name='Lac de Vouglans')


.. note:: Complex initialisation

    When a complex initialisation is required for a trait, a _XXX_default magic
    method can be implemented. It will be lazily called when trying to access
    the XXX trait. For example::

        def _name_default(self):
            """ Complex initialisation of the reservoir name. """

            return 'Undefined'

Validation
----------

Every trait does validation when the user tries to set its content:

::

    reservoir = Reservoir(name='Lac de Vouglans', max_storage=605)

    reservoir.max_storage = '230'
    ---------------------------------------------------------------------------
    TraitError                                Traceback (most recent call last)
    /Users/dpinte/projects/scipy-lecture-notes/advanced/traits/<ipython-input-7-979bdff9974a> in <module>()
    ----> 1 reservoir.max_storage = '230'

    /Users/dpinte/projects/ets/traits/traits/trait_handlers.pyc in error(self, object, name, value)
        166         """
        167         raise TraitError( object, name, self.full_info( object, name, value ),
    --> 168                           value )
        169
        170     def arg_error ( self, method, arg_num, object, name, value ):

    TraitError: The 'max_storage' trait of a Reservoir instance must be a float, but a value of '23' <type 'str'> was specified.

Documentation
-------------

By essence, all the traits do provide documentation about the model itself. The
declarative approach to the creation of classes makes it self-descriptive:

::

    from traits.api import HasTraits, Str, Float

    class Reservoir(HasTraits):

        name = Str
        max_storage = Float(100)


The **desc** metadata of the traits can be used to provide a more descriptive
information about the trait :

::

    from traits.api import HasTraits, Str, Float

    class Reservoir(HasTraits):

        name = Str
        max_storage = Float(100, desc='Maximal storage [hm3]')


Let's now define the complete reservoir class:

.. include:: reservoir.py
    :literal:

Visualisation
-------------

The Traits library is also aware of user interfaces and can pop up a default
view for the Reservoir class::

    reservoir1 = Reservoir()
    reservoir1.edit_traits()

.. image:: reservoir_default_view.png
    :align: center

TraitsUI simplifies the way user interfaces are created. Every trait on a
HasTraits class has a default editor that will manage the way the trait is
rendered to the screen (e.g. the Range trait is displayed as a slider, etc.).

In the very same vein as the Traits declarative way of creating classes,
TraitsUI provides a declarative interface to build user interfaces code:

.. include:: reservoir_simple_view.py
    :literal:

.. image:: reservoir_view.png
    :align: center


Deferral
--------

Being able to defer the definition of a trait and its value to another object
is a powerful feature of Traits.

.. include:: reservoir_state.py
   :literal:

A special trait allows to manage events and trigger function calls using the
magic **_xxxx_fired** method:

.. include:: reservoir_state_event.py
   :literal:

Dependency between objects can be made automatic using the trait **Property**.
The **depends_on** attribute expresses the dependency between the property and
other traits. When the other traits gets changed, the property is invalidated.
Again, Traits uses magic method names for the property :

    * _get_XXX for the getter of the XXX Property trait
    * _set_XXX for the setter of the XXX Property trait


.. include:: reservoir_state_property.py
   :literal:

.. note:: Caching property

    Heavy computation or long running computation might be a problem when
    accessing a property where the inputs have not changed. The
    @cached_property decorator can be used to cache the value and only
    recompute them once invalidated.

Let's extend the TraitsUI introduction with the ReservoirState example:

.. include:: reservoir_state_property_view.py
    :literal:

.. image:: reservoir_state_view.png
    :align: center

Some use cases need the delegation mechanism to be broken by the user when
setting the value of the trait. The **PrototypeFrom** trait implements this
behaviour.

.. include:: reservoir_turbine_prototype_from.py
    :literal:

Notification
------------

Traits implements a Listener pattern. For each trait a list of static and
dynamic listeners can be fed with callbacks. When the trait does change, all
the listeners are called.

Static listeners are defined using the _XXX_changed magic methods:

.. include:: reservoir_state_static_listener.py
    :literal:

The static trait notification signatures can be:

    * def _release_changed(self):
        pass
    * def _release_changed(self, new):
        pass
    * def _release_changed(self, old, new):
        pass
    * def _release_changed(self, name, old, new
        pass

.. note:: Listening to all the changes

    To listen to all the changes on a HasTraits class, the magic
    **_any_trait_changed** method can be implemented.

In many situations, you do not know in advance what type of listeners need to
be activated. Traits offers the ability to register listeners on the fly with
the dynamic listeners

.. include:: reservoir_state_dynamic_listener.py
    :literal:

The dynamic trait notification signatures are not the same as the static ones :

    * def wake_up_watchman():
        pass
    * def wake_up_watchman(new):
        pass
    * def wake_up_watchman(name, new):
        pass
    * def wake_up_watchman(object, name, new):
        pass
    * def wake_up_watchman(object, name, old, new):
        pass

Removing a dynamic listener can be done by:
    * calling the remove_trait_listener method on the trait with the listener
      method as argument,
    * calling the on_trait_change method with listener method and the keyword
      remove=True,
    * deleting the instance that holds the listener.

Listeners can also be added to classes using the **on_trait_change** decorator:

.. include:: reservoir_state_property_ontraitchange.py
    :literal:

The patterns supported by the on_trait_change method and decorator are
powerful. The reader should look at the docstring of HasTraits.on_trait_change
for the details.


Some more advanced traits
-------------------------

The following example demonstrate the usage of the Enum and List traits :

.. include:: reservoir_with_irrigation.py
    :literal:

Trait listeners can be used to listen to changes in the content of the list to
e.g. keep track of the total crop surface on linked to a given reservoir.

.. include:: reservoir_with_irrigation_listener.py
    :literal:

The next example shows how the Array trait can be used to feed a specialised
TraitsUI Item, the ChacoPlotItem:

.. include:: reservoir_evolution.py
    :literal:

.. image:: reservoir_evolution.png
    :align: center

References
==========

    * ETS repositories: http://github.com/enthought
    * Traits manual: http://github.enthought.com/traits/traits_user_manual/index.html
    * Traits UI manual: http://github.enthought.com/traitsui/traitsui_user_manual/index.html

    * Mailing list : enthought-dev@enthought.com
