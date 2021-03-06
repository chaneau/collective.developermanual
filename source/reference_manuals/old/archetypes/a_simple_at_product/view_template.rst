======================================
Adding a custom view for the content 
======================================

.. admonition:: Description

		Providing the custom presentation template for the
		InsantMessage, using Zope's browser layer mechanism. 

The browser layer concept
~~~~~~~~~~~~~~~~~~~~~~~~~

A browser layer is a concept introduced by Zope Component Architecture
(Zope 3), and which can be used in Plone. It is useful for registering
views and resources (images, CSS, JS) for the site, in a way that they
can override default elements (which are implicitly registered for the
default browser layer) or be overriden when needed, even through the
ZMI. *A browser layer is similar in purpose to a CMF skin layer, but is
implemented differently*.

To add a browser layer to your product, you need 3 steps:

-  Define the marker interface for the browser layer (for example,
   ``example.archetype.interfaces.IInstantMessageSpecific``.)
-  Add an XML file in your extension profile named ``browserlayer.xml``
   providing the browser layer settings to the site. *(This step is
   covered later as part of the various product setup details.)*
-  Register (using ZCML) your browser views, templates and resources.

For more about browser layer techniques, check `this tutorial`_.

Defining the browser layer interface
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add a marker interface for the browser layer (in ``interfaces.py``):

::

    from plone.theme.interfaces import IDefaultPloneLayer

    class IInstantMessageSpecific(IDefaultPloneLayer):
        """Marker interface that defines a Zope 3 skin layer for this product.
        """

Adding and registering the browser template
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To provide a custom view template for your content type, you need a page
template called ``instantmessage.pt`` in the ``browser/`` directory, and
a ZCML declaration in the ``configure.zcml`` to associate the template
to the ``IInstantMessageSpecific`` Zope 3 skin layer.

::

    <configure xmlns="http://namespaces.zope.org/zope"
        xmlns:browser="http://namespaces.zope.org/browser"
        i18n_domain="example.archetype" >
        <browser:page
            for="example.archetype.interfaces.IInstantMessage"
            layer="example.archetype.interfaces.IInstantMessageSpecific"
            name="instantmessage_view"
            template="instantmessage.pt"
            permission="zope2.View"
            />
    </configure>

Here is the example template code:

::

    <html metal:use-macro="here/main_template/macros/master"
          i18n:domain="plone" >
    <body>

    <div metal:fill-slot="main"
         tal:define="priority here/getPriority;
                     priority_color python:(priority == 'high' and 'red') or (priority == 'low' and 'green') or ''" >

            <h1 tal:content="context/Title"
                tal:attributes="style string:color:$priority_color" >
              Title
            </h1>

            <p tal:content="structure here/getBody" />          

            <div class="documentByLine">
              Message by <span tal:content="context/Creator" /> 
              with <strong tal:content="priority" /> priority.
              -
              <span tal:replace="python:here.toLocalizedTime(context.CreationDate(),long_format=1)" />
            </div>
        
    </div>

    </body>
    </html>

**Python notes:**

-  The new methods we use on the content object (getPriority, getBody,
   etc), called the “accessors”, are generated by Archetypes as part of
   its internal mechanisms, based on the field definition in the content
   schema; so if the field is called ‘priority’, there is a generated
   method called ‘getPriority’ responsible to return the stored value on
   the object. Note that the code of the method is not available
   somewhere for modification ; “generated” here means it is available
   in the server’s memory, within Archetypes engine’s registries, when
   the Zope server has started.

After the product installation step, which we still have to discuss (see
later), Plone should be able to find this template and use it as the
content object’s default view when you invoke the content’s URL.

.. _this tutorial: ../../../tutorial/customization-for-developers
