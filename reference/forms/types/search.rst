.. index::
   single: Form; Campi; search

Tipo di campo search
====================

Campo reso come ``<input type="search" />``, che è un campo testuale con speciali
funzionalità supportate da alcuni browser.

Maggiori informazioni su `DiveIntoHTML5.info`_

+---------------+----------------------------------------------------------------------+
| Reso come     | campo ``input search``                                               |
+---------------+----------------------------------------------------------------------+
| Opzioni       | - `max_length`_                                                      |
| ereditate     | - `required`_                                                        |
|               | - `label`_                                                           |
|               | - `trim`_                                                            |
|               | - `read_only`_                                                       |
|               | - `disabled`_                                                        |
|               | - `error_bubbling`_                                                  |
|               | - `mapped`_                                                          |
+---------------+----------------------------------------------------------------------+
| Tipo genitore | :doc:`text</reference/forms/types/text>`                             |
+---------------+----------------------------------------------------------------------+
| Classe        | :class:`Symfony\\Component\\Form\\Extension\\Core\\Type\\SearchType` |
+---------------+----------------------------------------------------------------------+

Opzioni ereditate
-----------------

Queste opzioni sono ereditate dal tipo :doc:`form</reference/forms/types/form>`:

.. include:: /reference/forms/types/options/max_length.rst.inc

.. include:: /reference/forms/types/options/required.rst.inc

.. include:: /reference/forms/types/options/label.rst.inc

.. include:: /reference/forms/types/options/trim.rst.inc

.. include:: /reference/forms/types/options/read_only.rst.inc

.. include:: /reference/forms/types/options/disabled.rst.inc

.. include:: /reference/forms/types/options/error_bubbling.rst.inc

.. include:: /reference/forms/types/options/mapped.rst.inc

.. _`DiveIntoHTML5.info`: http://diveintohtml5.info/forms.html#type-search
