Using Themes
============

**NOTE:** Themes made for PufferPanel 2.x versions will **not** function on PufferPanel 3.x versions

To install a theme on your PufferPanel instance make sure that the web file location defined in the
config (usually ``/var/www/pufferpanel``) actually exists (or create it if it doesn't). Make sure
that within that directory another directory named ``theme`` exists (or, again, create it if it
doesn't). Then place your theme file in that ``theme`` directory and make sure that PufferPanel can
read the web files by running ``chown -R pufferpanel:pufferpanel /path/to/web/files`` (replace
``/path/to/web/files`` with the actual path to the web files as defined in the config). When you now
navigate to your PufferPanel installation in your browser and log in you should be able to find your
newly installed theme in the theme selection on both, your account preferences page, as well as the
panels settings page.
