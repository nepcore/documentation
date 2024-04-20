Creating Themes
===============

Starting with version 3.0.0 PufferPanel offers a powerful theming system that allows you to apply extensive
customizations or create your own look and feel from scratch. This guide will help you getting started with
creating custom themes, however this guide will **NOT** attempt to teach you CSS, you can learn more about that
using for example the excellent resouces from `W3Schools <https://www.w3schools.com/Css/>`_. You will also need
a basic understanding of the JSON format.

Anatomy of a theme
------------------

A theme always consists of an uncompressed TAR archive using the ``.tar`` file extension, similar file extensions
like ``.tar.gz`` etc are compressed forms of TAR archives and **not** supported (note: just renaming such files does
not remove the compression and therefore does not result in a working theme). The name of this TAR file (minus the
file extension) will be used as the theme name. Inside this archive PufferPanel expects to find at least two specific
files, a ``manifest.json`` and a ``theme.css``, without these your theme will not function, any other files included
within the archive will be treated as assets, more on that later.

Manifest
--------

The ``manifest.json`` file contains some information about your theme and how it should function, it is formatted in
JSON and should always contain a JSON object at the root. Various fields within this object configure aspects of how
PufferPanel handles your theme.

+------------------------+---------+---------+-----------+-------------------------------------------------------------------------------------------+
| field                  | version | type    | default   | description                                                                               |
+========================+=========+=========+===========+===========================================================================================+
| ``keepDefaultCss``     | ≥ 3.0.0 | boolean | ``false`` | Whether to keep the default themes CSS.                                                   |
|                        |         |         |           | Keeping this ``false`` lets you write all styles completely from scratch.                 |
|                        |         |         |           | Setting this to ``true`` allows you to work off of what the default theme already offers. |
|                        |         |         |           | Your themes CSS will always be inserted after the default CSS and the default CSS uses    |
|                        |         |         |           | the ``!important`` modifier sparingly to make it easy to overwrite styles as needed.      |
+------------------------+---------+---------+-----------+-------------------------------------------------------------------------------------------+
| ``sidebarClosedBelow`` | ≥ 3.0.0 | number  | ``1200``  | Screens with a width smaller than what is defined here will default to having the         |
|                        |         |         |           | navigation sidebar closed, the user will need to activate it themselves.                  |
|                        |         |         |           | On larger screens the navigation sidebar will be open by default.                         |
|                        |         |         |           | Only applies to the JS logic for the navigation toggle button, your CSS may provide       |
|                        |         |         |           | other special handling e.g. using media queries as you see fit.                           |
+------------------------+---------+---------+-----------+-------------------------------------------------------------------------------------------+
| ``settings``           | ≥ 3.0.0 | object  | ``{}``    | This object lets you define settings through which users may customize aspects of your    |
|                        |         |         |           | theme.                                                                                    |
+------------------------+---------+---------+-----------+-------------------------------------------------------------------------------------------+

Settings
^^^^^^^^

The ``settings`` object in the ``manifest.json`` lets you expose customization settings to users of your theme, you are free to choose the keys
within this object as you see fit, each key should hold another object describing that setting, these objects must always have a ``type`` field,
any other fields that must or can be set depend on the ``type`` chosen. To make a setting actually understandable to a user it is **highly** recommended to
provide a name for the input, this can be done in one of three ways, you can use the ``tkey`` field to reference the translation files already included
in PufferPanel, you can use the ``tlabels`` key to provide an object with translated strings keyed by the locale or you can use the ``label`` key to
simply provide one fixed string as the name displayed with the input without any translation handling (should either of the first two be defined but not
produce a translated string for the users locale the ``label`` field may also be used as a fallback).

class
*****

If the ``type`` field is set to the value ``class`` PufferPanel will present to the user a dropdown with which they can choose between options you define
within this setting object. These options are defined using the ``options`` field, which should hold an array of objects, each of these objects should have
a label which can be defined in the same way as the name for the setting input and a ``value`` key which is a string of your choosing that will be added to the
applications root element as a CSS class allowing you to scope CSS rules based on which of these options is selected, additionally you can mark one of these options
as the default value by setting the ``default`` key to ``true``, if no option is explicit marked as default the first one will be used, the behaviour for
having multiple options marked as default is undefined and may or may not cause errors.

Example:

.. code-block:: json

    {
      "type": "class",
      "tkey": "common.theme.Mode",
      "options": [
        {
          "value": "mode-auto",
          "tkey": "common.theme.mode.Auto",
          "default": true
        },
        {
          "value": "mode-light",
          "label": "Light"
        },
        {
          "value": "mode-dark",
          "tlabels": {
            "en_US": "Dark",
            "de_DE": "Dunkel"
          }
        }
      ]
    }

color
*****

If the ``type`` field is set to ``color`` PufferPanel will present to the user a color picker with which they may choose any color. The ``default`` key defines
the color to use by default and the ``var`` field names a CSS variable to set to the user selected color. To aid in making designs capable of working with
a broad variety of user color choices the ``derive`` field, an array of objects, allows using the chosen color to automatically derive other colors. Color
derivation currently supports the types ``contrast``, ``opacity`` and ``hueShift``.

``contrast`` lets you provide a set of possible colors in the ``options`` array and picks the one with the best contrast to provide in the CSS variable named in
the ``var`` field. The ``foreground`` field should be set to ``true`` if the derived color will be used as foreground or ``false`` if it will be used as background,
mixed use of the same derived color is not recommended, instead consider using the ``contrast`` derivation multiple times to derive separate colors to account
for cases where colors may not work as well interchangably.

``opacity`` lets you calculate a partially transparent variation of the selected color, useful e.g. for use in hover effects, it simply takes a ``var`` field naming
the CSS variable to provide the output in and a key ``opacity`` defining how opaque or transparent the color should be, ``1`` for fully opaque, ``0`` for fully
transparent, decimals for anything in between.

``hueShift`` lets you generate other colors by changing the hue of the selected color without touching other aspects like the saturation or lightness. It takes a
``var`` field naming the CSS variable the resulting color will be provided in and a ``hueShift`` field that defines by how many degrees to shift the color on the
hue circle, negative values are allowed here, 360 is a full circle and wraps back around to the starting color.

Example:

.. code-block:: json

    {
      "type": "color",
      "var": "--primary",
      "default": "#07a7e3",
      "derive": [
        {
          "type": "contrast",
          "foreground": true,
          "options": ["#eee", "#333"],
          "var": "--text-primary"
        },
        {
          "type": "opacity",
          "opacity": 0.15,
          "var": "--primary-hover"
        },
        {
          "type": "hueShift",
          "hueShift": 180,
          "var": "--primary-complement"
        }
      ],
      "tkey": "common.theme.BaseColor"
    }

Assets
------

If you want to include assets like images or fonts you can simply place these in your themes TAR file, PufferPanel will extract these assets and make them
easily available to your CSS by their path within the archive.

Suppose your themes archive contains a file structure like the following:

.. code-block::

    My Theme.tar
    ├ manifest.json
    ├ theme.css
    └ img
      └ bg.png

You could now include a rule in your CSS like ``background-image: url('img/bg.png')`` and the image file will be used as a background for
the selector this rule appears on. The same concept works for any kind of resource.

Stylesheet
----------

The part of your theme that actually changes visuals, the CSS, lives in a ``theme.css`` file within the theme archive. While you are free to use CSS
preprocessors like SASS to make your theme, you will need to take care of running the preprocessor yourself as themes only support standard CSS
as supported by browsers.

There are also a few special tricks through which PufferPanel allows you to take control of aspects otherwise hard or impossible to deal with purely with CSS.

PufferPanel always injects a CSS variable named ``--inner-height``, this variable holds the height reported by the JS property ``window.innerHeight`` and gets
updated with any resize events, as this may be needed to deal with mobile browers' disappearing address bars.

You can take control of the theme the ace text editor is displayed in using the ``--ace-theme`` CSS variable, for example setting it to ``monokai`` will result
in the ace editor using the monokai theme

The component used to render graphs for server statistics doesn't respond well to direct styling via CSS, therefore the following CSS variables can be used:
  * ``--apex-font`` lets you set the font to be used
  * ``--apex-background-color`` controls the charts background color (note: this only supports hex colors, no color names or rgb() functions)
  * ``--apex-text-color`` controls the text color (note: this only supports hex colors, no color names or rgb() functions)
  * ``--apex-series-colors`` controls the colors of the graph lines, multiple colors can be given separated by comma (note: this only supports hex colors, no color names or rgb() functions)
