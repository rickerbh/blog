{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2008-01-30 21:33:50", :layout :post, :draft? false, :title "Time for the big upgrade"}

Righto, I've been putting it off long enough.  <a href="http://gotthegist.co.nz">Got the GiST</a> has been running on rails 1.1.6 almost since it has been launched, and I'd like to take advantage on some of that RESTful 2.0 goodness.  So, I've decided it's time to upgrade it.

I've got a couple of issues though.  I currently use (shhhh, I'm about to say a dirty word) <em>Engines</em> to handle my user logins and roles...  They don't work with much past rails 1.1.6.  I also don't have good admin screens on the application, and whenever I need to do maintenance (which is very rare) I need to go and edit the database directly :-(

So, there are a number of things that I want to do to the app.
<ol>
	<li>Upgrade to Rails 2.0.2</li>
	<li>Convert my user login goodies to use <a href="http://technoweenie.stikipad.com/plugins/show/Acts+as+Authenticated">acts_as_authenticated</a></li>
	<li>Get another way to handle roles</li>
	<li>Implement <a href="http://streamlinedframework.org/">Streamlined</a> for admin screens</li>
	<li>Write some tests - I've was a bit slack when I initially wrote <a href="http://gotthegist.co.nz">Got the GiST</a></li>
	<li>Refactor my code - this goes without saying - some of it is a bit shabby</li>
</ol>

The main issue I have is that I'm not sure what order I want to do this in.  The issue that I have is there are a number of dependencies.  For example, old-style Engines doesn't work past 1.1.6 - and I have my complete login and role system built around that.  <a href="http://streamlinedframework.org/">streamlined</a> don't appear to work with Rails 1.1.x.  <a href="http://technoweenie.stikipad.com/plugins/show/Acts+as+Authenticated">acts_as_authenticated</a> requires a simple bit of script hackery to work with 1.1.6, so that might be able to come early...  

I think what I'm going to do is drop Engines ASAP.  That's the main cause of my pain.  I need to recreate with acts_as_authenticated essentially the same methods for authentication as login_engine currently has, as well as find a (perhaps temporary) solution to my roles issues.  Then I'll set about writing tests.  Next step I think could be to upgrade to 1.2.6, and then implement <a href="http://streamlinedframework.org/">streamlined</a>. I think I'll then be ready for a migration to Rails 2.0.

Unless someone else has a much better suggestion!
