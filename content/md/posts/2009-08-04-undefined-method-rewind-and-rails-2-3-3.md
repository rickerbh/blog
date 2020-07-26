{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2009-08-04 19:06:29", :layout :post, :draft? false, :title "Undefined method rewind and rails 2.3.3"}

Today I updated a project I'm working on from rails 2.2.something to 2.3.3.  Then, I started getting the following wacky error.

<code>
undefined method `rewind' for #&lt;TCPSocket:0x3631e58&gt;
</code>

Turns out if you see this after a rails 2.3.3 update, you might just need a phusion passenger update (I was on passenger 2.1.3, apparently this error happens on passenger 2.1.2 too)

<code>
sudo gem update passenger
sudo passenger-install-apache2-module
</code>

Put the lines recommended by the install script in your apache config file, restart apache and you're good to go.

Hope this helps someone.


