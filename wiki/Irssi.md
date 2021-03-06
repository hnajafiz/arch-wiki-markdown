Irssi
=====

Related articles

-   IRC channels
-   IRC Channel

irssi is a modular, ncurses based IRC (Internet Relay Chat) client for
UNIX systems. It also supports SILC and ICB protocols via plugins.

Contents
--------

-   1 Installation
-   2 Configuration
    -   2.1 Auto-logging
    -   2.2 Auto-connect to #archlinux on startup
    -   2.3 Hide joins, parts, and quits
-   3 Basic usage
    -   3.1 Commands
-   4 Script installation
-   5 Freenode SSL and SASL installation
    -   5.1 SSL Connection
        -   5.1.1 Client certificates
    -   5.2 Authenticating with SASL
-   6 HTTP Proxy
-   7 irssi with nicklist in tmux
-   8 Virtual hostname (vhost)
    -   8.1 Required preconfigurations
    -   8.2 Enabling the vhost
-   9 See also

Installation
------------

Install irssi, available in the official repositories.

You may also want to install irssi-systemd.

Configuration
-------------

Personal configuration file should be located at ~/.irssi/config. You
can start irssi with an alternate config file using the --config flag.

-   You can use /save to save your current configuration to the config
    file.

-   You can save the location of your currently opened windows by
    entering /layout save

-   It sometimes might happen that umlauts are not correctly displayed.
    To fix this problem you have to set the right encoding with the
    following commands directly in irssi.

    /set recode_autodetect_utf8 ON 
    /set recode_fallback CP1252
    /save

> Auto-logging

    /SET autolog ON
    /save

> Auto-connect to #archlinux on startup

Start irssi and then type the following in it:

    /server add -auto -network fn irc.freenode.net

fn is a common abbreviation for the freenode network, but any preferred
word can be substituted for it (e.g. foo).

Now to automatically identify your nick for a given password, type:

    /network add -nick user -autosendcmd "/msg nickserv IDENTIFY *******" fn

where user is the nick with which you registered to nickserv and *******
is the password for that nick, replace fn with the word that you used in
the first command (e.g foo) if that is the case.

Note:Password will be visible when you type it and also it can be seen
in ~/.irssi/config, so you can omit this step if you want to.

Note:If your password contain
"" (dolar symbol), you have to prefix it with another "". So it would be
"$$"

    /channel add -auto #archlinux fn
    /channel add -auto #archlinux-offtopic fn

Note:Instead of using passwords with NickServ, you can authenticate
yourself using certificates on SSL connections as described in #Client
certificates

> Hide joins, parts, and quits

In order to ignore showing of joining, leaving and quiting of users for
all channels type the following in irssi:

    /ignore * joins
    /ignore * parts
    /ignore * quits

Basic usage
-----------

Note:This section assumes you already know the basics of IRC and have
used other clients in the past. For a more detailed introduction check
the official documentation.

To start irssi issue the following command in a terminal:

    $ irssi

Many people prefer to run irssi within a terminal multiplexer since some
scripts like the nicklist.pl script are dependent on a secondary window.
Additionally, it allows the user to easily disconnect and reconnect to a
session. Therefore, it is recommended that you select a multiplexer
(e.g. GNU Screen or tmux) and review how it functions.

> Commands

> Connection

/server

/s

These change the server of the current network.

/connect

/c

These open a new connection to a server. This is what you want to use in
order to connect to multiple servers simultaneously (Ctrl+X switches
between multiple servers).

/disconnect

/dc

These close the current connection to a server.

> Movement

ALT+(1-0,q-p,etc)

Changes the currently active window. Or use Ctrl+n for the next window
or Ctrl+p for the previous window.

/window 1

/w 1

Takes you to the first window. Windows go from are numbered across the
top of your keyboard (1-0) and then start on the next row down (q-p).

/window close

/wc

These close the current window.

/window move 1

/w move 1

These move the current window to the first window position.

/layout save

This will save the current window positions for the next time you start
irssi.

> Miscellaneous

/set

This shows a list of all your current settings.

/help

This provides a helpful description/explanation for whatever parameter
provided.

/alias

Lets you create your own shortcuts.

Script installation
-------------------

As an example, this section will outline the installation of a spell
checking script.

Install ispell, an interactive spell-checking program for Unix.

Create a directory to hold your irssi scripts and within that, a
directory that contains scripts which will be automatically run when
starting irssi:

    $ mkdir -p ~/.irssi/scripts/autorun

Download the irssi spell-checking script, spell.pl into the script
directory:

    $ cd ~/.irssi/scripts
    $ wget http://scripts.irssi.org/scripts/spell.pl .

As root run the following command (this will download perl module to
/usr/share/perl5/site_perl/Lingua/Ispell.pm):

    # perl -MCPAN -e 'install Lingua::Ispell'

As a root change the path in ispell.pm to:

    /usr/bin/ispell

If you do not want to use CPAN review [1].

Start irssi and load the spell-checking script:

    /script load spell.pl

    - - Irssi: Loaded script spell

Bind Alt + s to spell check your current line.

    /bind meta-s /_spellcheck

If you want to autorun the script when you start irssi, just link the
script into the autorun folder:

    $ cd ~/.irssi/scripts/autorun/
    $ ln -s ../spell.pl .

Freenode SSL and SASL installation
----------------------------------

> SSL Connection

Freenode uses port 6697, 7000 and 7070 for SSL connections. To connect
to Freenode IRC network via SSL you have to setup an new connection.
Start irssi and paste:

     /server add -auto -ssl -ssl_verify -ssl_capath /etc/ssl/certs -network freenode irc.freenode.net 6697

Save your new settings with:

    /save

If everything works you will see the "Z" mode set. It should look like
this: "Mode change (+Zi) for user your-nick"

Client certificates

Freenode and OFTC support authentication using SSL certificates,
providing an alternative to plaintext passwords. See Freenode's
Identifying with CERTFP for more extensive details.

To create an password-less certificate that is valid for 730 days (when
requested to enter details like state or even Common Name (CN), you can
fill anything you want):

    openssl req -newkey rsa:2048 -days 730 -x509 -keyout irssi.key -out irssi.crt -nodes 
    cat irssi.crt irssi.key > ~/.irssi/irssi.pem
    chmod 600 ~/.irssi/irssi.pem
    rm irssi.crt irssi.key

In irssi, disconnect from the network and add the client certificate and
keys:

    /disconnect Freenode
    /server add -ssl_cert ~/.irssi/irssi.pem irc.freenode.net

Now connect (not /reconnect) and register your fingerprint

    /connect Freenode
    /msg NickServ identify YOUR_PASSWORD
    /msg NickServ cert add

At this point, you can remove your password from the configuration file
(if you saved it in there) and save your config with:

    /save

> Authenticating with SASL

Install the SASL script irssi-script-sasl from AUR. Load the script with
each start of irssi:

    cd ~/.irssi/scripts/
    cd autorun
    ln -s ../cap_sasl.pl cap_sasl.pl

Then open irssi and run the SASL script and enter your SASL settings.
Please use your main nick "your-primary-nick" and "your-password" which
is registered with nickserv.:

    /RUN cap_sasl.pl
    /sasl set freenode your-primary-nick your-password DH-BLOWFISH
    /sasl save
    /save

Restart irssi and have a look for "Irssi: SASL authentication
successful".

HTTP Proxy
----------

To use irssi behind a HTTP proxy, the following commands are required:

    /SET use_proxy ON
    /SET proxy_address <Proxy host address>
    /SET proxy_port <Proxy port>
    /SET -clear proxy_string
    /SET proxy_string_after conn %s %d
    /EVAL SET proxy_string CONNECT %s:%d HTTP/1.0\n\n

irssi should then alter its config file correspondingly; if the proxy is
not required, just set use_proxy to OFF.

Should the proxy require a password, try:

    /SET proxy_password your_pass

Otherwise:

    /SET -clear proxy_password

Note:SSL behind a proxy will fail with these settings.

irssi with nicklist in tmux
---------------------------

The irssi plugin 'nicklist' offers to add a pane listing the users on
the channel currently viewed. It has two methods to do this:

-   screen, which simply adds the list to the right of irssi, but brings
    the disadvantage that the entire window gets redrawn every time
    irssi prints a line.

-   fifo, which like the name suggests writes the list into a fifo that
    can then be continuously read with e. g. cat ~/.irssi/nicklistfifo.

nicklist will use the more efficient fifo with:

    /NICKLIST FIFO

This fifo can be used in a tmux window split vertically with irssi in
its left pane and the cat from above in a small one in its right. Since
the pane is dependent on its creating tmux session's geometry, a
subsequent session with a different one needs to recreate it (which also
implies a switch in irssi windows to refill the fifo).

E. g., the following script first checks for a running irssi, presumed
to have been run by a previous execution of itself. Unless found it
creates a new tmux session, a window named after and running irssi and
then the pane with cat. If however irssi was found it merely attaches to
the session and recreates the cat pane.

    #!/bin/bash

    T3=$(pgrep -u $USER -x irssi)

    irssi_nickpane() {
        tmux setw main-pane-width $(( $(tput cols) - 21));
        tmux splitw -v "cat ~/.irssi/nicklistfifo";
        tmux selectl main-vertical;
        tmux selectw -t irssi;
        tmux selectp -t 0;
    }

    irssi_repair() {
        tmux selectw -t irssi
        (( $(tmux lsp | wc -l) > 1 )) && tmux killp -a -t 0
        irssi_nickpane
    }

    if [ -z "$T3" ]; then
        tmux new-session -d -s main;
        tmux new-window -t main -n irssi irssi;
        irssi_nickpane ;
    fi
        tmux attach-session -d -t main;
        irssi_repair ;
    exit 0

Virtual hostname (vhost)
------------------------

A vhost can be used to change your hostname when connected to an
IRC-server, commonly viewed when joining/parting or doing a whois. This
is most commonly done on a server which have a static IP address.
Without a vhost it would commonly look like so when doing a 'whois':

    nick@123.456.78.90.isp.com

The result of a successfull vhost could be like so if you have the
domain example.com available:

    nick@example.com

Keep in mind that not every IRC-server supports the use of vhost. This
might be individually set between the servers and not the network, so if
you're experiencing issues with one server try another on the same
network.

> Required preconfigurations

irssi supports using a vhost as long as the required configurations has
been set. This includes especially that your host supports Recursive DNS
Lookup (rDNS) using Pointer record (PTR). Additionally you should add an
appropriate line to your /etc/hosts file.

To see if this is working, test with the 'host' DNS lookup utility
included in dnsutils like so (where ip is a normal IPv4 address):

    host ip

If this returns something in the lines of this then you know that your
rDNS is working.

    ip.in-addr.arpa domain name pointer example.com

> Enabling the vhost

There are a couple of ways to connect to a server with a given hostname.
One is using the 'server' command with a -host argument like so:

    /server -host example.com irc.freenode.org

Another way would be to set your hostname (vhost) with the 'set' command
which will save your hostname to ~/.irssi/config:

    /set hostname example.com
    /save
    /server irc.freenode.org

See also
--------

-   Official website
-   Setting up Irssi - post on helpful Linux tidbits
-   Guide to efficiently using Irssi and screen by Matt Sparks
-   Official List of Irssi scripts
-   IRC notifications with dzen2 by Jason Ryan
-   Irssi’s /channel, /network, /server and /connect – What it means by
    Aaron Toponce

Retrieved from
"https://wiki.archlinux.org/index.php?title=Irssi&oldid=305973"

Category:

-   Internet Relay Chat

-   This page was last modified on 20 March 2014, at 17:32.
-   Content is available under GNU Free Documentation License 1.3 or
    later unless otherwise noted.
-   Privacy policy
-   About ArchWiki
-   Disclaimers
