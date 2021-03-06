About ``ntfy``
==============
|Version|_ |Docs|_ |Build|_ |WinBuild|_ |Coverage|_ |Requires|_

.. |Version| image:: https://img.shields.io/pypi/v/ntfy.svg
.. _Version: https://pypi.python.org/pypi/ntfy
.. |Docs| image:: http://readthedocs.org/projects/ntfy/badge/?version=latest
.. _Docs: http://ntfy.readthedocs.org/en/stable/?badge=latest
.. |Build| image:: https://travis-ci.org/dschep/ntfy.svg?branch=master
.. _Build: https://travis-ci.org/dschep/ntfy
.. |WinBuild| image:: https://ci.appveyor.com/api/projects/status/fw6oycy7px0k23gi/branch/master?svg=true
.. _WinBuild: https://ci.appveyor.com/project/dschep/ntfy
.. |Coverage| image:: https://coveralls.io/repos/github/dschep/ntfy/badge.svg?branch=master
.. _Coverage: https://coveralls.io/github/dschep/ntfy?brach=master
.. |Requires| image:: https://requires.io/github/dschep/ntfy/requirements.svg?branch=master
.. _Requires: https://requires.io/github/dschep/ntfy/requirements/?branch=master


``ntfy`` brings notification to your shell. It can automatically provide
desktop notifications when long running commands finish or it can send
push notifications to your phone when a specific command finishes.
Confused? This video demonstrates some of this functionality:

.. image:: https://raw.githubusercontent.com/dschep/ntfy/master/docs/demo.gif

Quickstart
----------

.. code:: shell

    $ sudo pip install ntfy
    $ ntfy send test
    # send a notification when the command `sleep 10` finishes
    # this sends the message '"sleep 10" succeeded in 0:10 minutes'
    $ ntfy done sleep 10
    $ ntfy -b pushover -o user_key t0k3n send 'Pushover test!'
    $ ntfy -t 'ntfy' send "Here's a custom notification title!"
    $ echo -e 'backends: ["pushover"]\npushover: {"user_key": "t0k3n"}' > ~/.ntfy.yml
    $ ntfy send "Pushover via config file!"
    $ ntfy done --pid 6379  # pid extra
    $ ntfy send ":tada: ntfy supports emoji! :100:"  # emoji extra
    # Enable shell integration
    $ echo 'eval "$(ntfy shell-integration)"' >> ~/.bashrc

Install
-------
The install technique in the quickstart is the suggested method of installation.
It can be installed in a virtualenv, but with some caveats: Linux notifications
require ``--system-site-packages`` for the virtualenv and OS X notifications
don't work at all.

Shell integration
~~~~~~~~~~~~~~~~~
``ntfy`` has support for **automatically** sending notifications when long
running commands finish in bash and zsh. In bash it emulates zsh's preexec and
precmd functionality with `rcaloras/bash-preexec <https://github.com/rcaloras/bash-preexec>`_.
To enable it add the following to your ``.bashrc`` or ``.zshrc``:

.. code:: shell

    eval "$(ntfy shell-integration)"

By default it will only send notifications for commands lasting longer than 10
seconds and if the terminal is focused. Terminal focus works on X11(Linux) and
with Terminal.app and iTerm2 on MacOS. Both options can be configured via the
``--longer-than`` and ``--foreground-too`` options.

To avoid unnecessary notifications when running interactive programs, programs
listed in ``AUTO_NTFY_DONE_IGNORE`` don't generate notifications. For example:

.. code:: shell

    export AUTO_NTFY_DONE_IGNORE="vim screen meld"

Extras
~~~~~~
``ntfy`` has a few features that require extra dependencies.
    * ``ntfy done -p $PID`` requires installing as ``ntfy[pid]``
    * emjoi support requires installing as ``ntfy[emoji]``
    * XMPP support requires installing as ``ntfy[xmpp]``
    * Telegram support requires installing as ``ntfy[telegram]``
    * Instapush support requires installing as ``ntfy[instapush]``
    * Slack support requires installing as ``ntfy[slack]``

To install multiple extras, separate with commas: e.g., ``ntfy[pid,emjoi]``.

Configuring ``ntfy``
--------------------

``ntfy`` is configured with a YAML file stored at ``~/.config/ntfy/ntfy.yml``

Backends
~~~~~~~~

The backends key specifies what backends to use by default. Each backend has
its own configuration, stored in a key of its own name. For example:

.. code:: yaml

    ---
    backends:
        - pushover
        - simplepush
        - slack
        - linux
        - xmpp
    pushover:
        user_key: hunter2
    pushbullet:
        access_token: hunter2
    simplepush:
        key: hunter2
    slack:
        token: slacktoken
        recipient: "#slackchannel"
    xmpp:
         jid: "user@gmail.com"
         password: "xxxx"
         mtype: "chat"
         recipient: "me@jit.si"

If you want mulitple configs for the same backend type, you can specify any
name and then specify the backend with a backend key. For example:

.. code:: yaml

    ---
    pushover:
        user_key: hunter2
    cellphone:
        backend: pushover
        user_key: hunter2

See the backends below for available backends and options.

`Pushover <https://pushover.net>`_ - ``pushover``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Required parameters:
    * ``user_key``

Optional parameters:
    * ``sound``
    * ``priority``
    * ``expire``
    * ``retry``
    * ``callback``
    * ``api_token`` - use your own application token
    * ``device`` - target a device, if omitted, notification is sent to all devices
    * ``url``
    * ``url_title``
    * ``html``

`Pushbullet <https://pushbullet.com>`_ - ``pushbullet``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Required parameter:
    * ``access_token`` - Your Pushbullet access token, created at https://www.pushbullet.com/#settings/account

Optional parameters:
    * ``device_iden`` - a device identifier, if omited, notification is sent to all devices
    * ``email`` - send notification to pushbullet user with the specified email or send an email if they aren't a pushullet user

`Simplepush <https://simplepush.io>`_ - ``simplepush``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Required parameter:
    * ``key`` - Your Simplepush key, created by installing the Android App (no registration required) at https://simplepush.io

XMPP - ``xmpp``
~~~~~~~~~~~~~~~
Requires parameters:
    * ``jid``
    * ``password``
    * ``recipient``
Optional parameters
    * ``hostname`` (if not from jid)
    * ``port``
    * ``path_to_certs``
    * ``mtype`` ('chat' required for Google Hangouts)

Requires extras, install like this: ``pip install ntfy[xmpp]``.

To verify the SSL certificates offered by a server:
path_to_certs = "path/to/ca/cert"

Without dnspython library installed, you will need
to specify the server hostname if it doesn't match the jid.

For example, to use Google Talk you would need to use:
hostname = 'talk.google.com'

Specify port if other than 5222.
NOTE: Ignored without specified hostname

`Telegram <https://telegram.org>`_ - ``telegram``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Requires extras, install like this: ``pip install ntfy[telegram]``.

Requires ``ntfy`` to be installed as ``ntfy[telegram]``. This backend is
configured the first time you will try to use it: ``ntfy -b telegram send
"Telegram configured for ntfy"``.

`Pushjet <https://pushjet.io/>`_ - ``pushjet``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Required parameter:
    * ``secret`` - The Pushjet service secret token, created with http://docs.pushjet.io/docs/creating-a-new-service

Optional parameters:
    * ``endpoint`` - custom Pushjet API endpoint
        (defaults to https://api.pushjet.io)
    * ``level`` - The importance level from 1(low) to 5(high)
    * ``link``

`Notifico <https://n.tkte.ch/>`_ - ``notifico``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Required parameter:
    * ``webhook`` - The webhook link, created at https://n.tkte.ch/
                    (choose ``Plain Text`` service when creating the webhook)

`Slack <https://slack.com>`_ - ``Slack``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Requires extras, install like this: ``pip install ntfy[slack]``.

Required parameter:
    * ``token`` - The Slack service secret token, created in https://api.slack.com/web#authentication
    * ``recipient`` - The Slack channel or user to send notifications to. If you use the ``#`` symbol the message is send to a Slack channel and if you use the ``@`` symbol the message is send to a Slack user.

`Instapush <https://instapush.im/>`_ - ``insta``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Requires extras, install like this ``pip install ntfy[instapush]``.

Instapush does not support notification title.
It sends template-driven notifications, so you have to setup you events on the dashboard first.
The backend is called insta due to homonymy with the instapush python wrapper

Required parameters:
    * ``appid`` - The application id
    * ``secret`` - The application secret
    * ``event_name`` - The instapush event to be used
    * ``trackers`` - The array of trakers to use

Note on trackers:
Trackers are placeholders for events (a sort of notification template). If you defined more than one tracker in your event
you'll have to provide more messages. At the moment, the only way to do so is to separate each message with a colon (:) character.
You can also escape the separator character:
Example:

.. code:: shell

    ntfy -b insta send "message1:message2"
    ntfy -b insta send "message1:message2\:with\:colons"

`Prowl <https://www.prowlapp.com/>`_ - ``prowl``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Optional parameters:
    * ``api_key``
    * ``provider_key``
    * ``priority``
    * ``url``

`Linux Desktop Notifications <https://developer.gnome.org/notification-spec/>`_ - ``linux``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Works via `dbus`, works with most DEs like Gnome, KDE, XFCE and with libnotify.

The following dependecies should be installed.

.. code:: shell

    $ sudo apt-get install libdbus-glib-1-dev libdbus-1-dev
    $ pip install --user dbus-python

You will need to install some font that supports emojis (in Debian `fonts-symbola` or Gentoo `media-fonts/symbola`).

Windows Desktop Notifications - ``win32``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Uses ``pywin32``.

Mac OS X Notification Center - ``darwin``
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
Requires ``ntfy`` to be installed globally (not in a virtualenv).

System log - ``systemlog``
~~~~~~~~~~~~~~~~~~~~~~~~~~
Uses the ``syslog`` core Python module, which is not available on Windows
platforms.

Optional parameters:
    * ``prio`` - Syslog priority level.  Default is ``ALERT``.  Possible values
      are:

      * EMERG
      * ALERT
      * CRIT
      * ERR
      * WARNING
      * NOTICE
      * INFO
      * DEBUG

    * ``facility`` - Syslog facility.  Default is ``LOCAL5``.  Possible values
      are:

      * KERN
      * USER
      * MAIL
      * DAEMON
      * AUTH
      * LPR
      * NEWS
      * UUCP
      * CRON
      * SYSLOG
      * LOCAL0
      * LOCAL1
      * LOCAL2
      * LOCAL3
      * LOCAL4
      * LOCAL5
      * LOCAL6
      * LOCAL7

    * ``fmt`` - Format of the message to be sent to the system logger.  The
      title and the message are specified using the following placeholders:

      * ``{title}``
      * ``{message}``

      Default is ``[{title}] {message}``.

Other options
~~~~~~~~~~~~~

Title is configurable with the `title` key in the config. Example:

.. code:: yaml

    ---
    title: Customized Title


Backends ToDo
~~~~~~~~~~~~~
-  `Airgram <http://www.airgramapp.com>`_
-  `Pushalot <https://pushalot.com>`_
-  `Boxcar <https://boxcar.io>`_

Testing
-------

.. code:: shell

    python setup.py test

Contributors
------------
- `dschep <https://github.com/dschep>`_ - Maintainer & Lead Developer
- `danryder <https://github.com/danryder>`_ - XMPP Backend & emoji support
- `oz123 <https://github.com/oz123>`_ - Linux desktop notification improvements
- `schwert <https://github.com/schwert>`_ - PushJet support
- `rahiel <https://github.com/rahiel>`_ - Telegram support
- `tymm <https://github.com/tymm>`_ - Simplepush support
- `jungle-boogie <https://github.com/jungle-boogie>`_ - Documentation updates
- `tjbenator <https://github.com/tjbenator>`_ - Advanced Pushover options
- `mobiusklein <https://github.com/mobiusklein>`_ - Win32 Bugfix
- `rcaloras <https://github.com/rcaloras>`_ - Creator of `bash-prexec`, without which there woudn't be bash shell integration for `ntfy`
- `eightnoteight <https://github.com/eightnoteight>`_ - Notifico support
- `juanpabloaj <https://github.com/juanpabloaj>`_ - Slack support
- `giuseongit <https://github.com/giuseongit>`_ - Instapush support
- `jlesage <https://github.com/jlesage>`_ - Systemlog support
- `sambrightman <https://github.com/sambrightman>`_ - Prowl support
