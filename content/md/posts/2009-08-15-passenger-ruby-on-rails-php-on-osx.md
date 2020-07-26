{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2009-08-15 10:06:53", :layout :post, :draft? false, :title "Passenger (Ruby on Rails) + PHP on OSX"}

I've spent the last hour or so trying various things out to get passenger and PHP to play nicely together on my mac under OS X (Leopard) and apache2.

The situation I was finding was that PHP apps would run, but only if you explicitly call the script (ie <tt>index.php</tt>) rather than just the directory.  If you called the directory, passenger would take over and give me a rails routing error.

The issue was to do with the passenger vhosts configuration.  On my machine I have an number of ruby on rails apps configured with the passenger preferences pane (creating vhost entries within <tt>/private/etc/apache2/passenger_pane_vhosts/</tt>.  I have enabled user_dirs, so that the users of my machine's pages (and other apps) are served from their <tt>~username/Sites</tt> directory.

My users configuration info for apache is installed in <tt>/private/etc/apache2/users/</tt>, and the instructions to load the configuration from that directory is stored within <tt>/private/etc/apache2/extra/httpd-userdir.conf</tt> (content below).

```
# Settings for user home directories
#
# Required module: mod_userdir
#
# UserDir: The name of the directory that is appended onto a user's home
# directory if a ~user request is received.  Note that you must also set
# the default access control for these directories, as in the example below.
#
UserDir Sites
#
# Users might not be in /Users/*/Sites, so use user-specific config files.
#
Include /private/etc/apache2/users/*.conf
```

To get everything working together nicely, I merely wrapped this inside a vhosts configuration directive, and gave it a ServerName of localhost - so that this vhost would be the one that responds to requests for localhost, rather than some random passenger vhost assuming it was the boss of everything.  New <tt>/private/etc/apache2/extra/httpd-userdir.conf</tt> below.

```
&lt;VirtualHost *:80&gt;
  ServerName localhost
  UserDir Sites
  Include /private/etc/apache2/users/*.conf
&lt;/VirtualHost&gt;
```

Thanks to this, all of my rails apps are served under passenger, and I can have static HTML, PHP and camping apps (previously configured - nothing to do with the above) all served from within my <tt>~username/Sites</tt> directory.

Hope this helps someone.


