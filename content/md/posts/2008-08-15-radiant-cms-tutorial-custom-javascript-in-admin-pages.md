{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2008-08-15 19:00:45", :layout :post, :draft? false, :title "Radiant CMS Tutorial - Custom Javascript in Admin Pages"}

At the moment I'm working on a fairly complex extension for <a href="http://radiantcms.org/" target="_blank">Radiant CMS</a>. In creating the administrative pages I want to use a javascript library that is not distributed with the core radiant code. This post will describe how to implement your own extension that can use an external javascript library, without playing around with any of the core radiant files to inject the javascript into the administrative layout.

It's actually really easy to do.  If you haven't created a Radiant extension before I'd recommend following <a href="http://wiki.radiantcms.org/Creating_Radiant_Extensions" target="_blank">this tutorial</a> on the <a href="http://wiki.radiantcms.org" target="_blank">Radiant wiki</a>.

Inside the Radiant GEM, the standard page layout resides at app/views/layouts/application.html.haml

The lines of code that insert the javascript tags are:

``` ruby
- @javascripts.uniq.each do |javascript|
  = javascript_include_tag javascript
```

The @javascripts variable is populated from inside the app/controllers/application_controller.rb file. The culprit is below.

``` ruby
def include_javascript(script)
  @javascripts &lt;&lt; script
end
```

Pretty simple huh? So, all you need to do, is call this method from within your new controller, because all controllers inherit from the application_controller.rb file, so they have access to this method, and you can have different javascript included for each method if you so wish. In (assuming the example LinkRoll extension was built as linked above) vendor/extensions/link_roll/app/controllers/admin/links_controller.rb

``` ruby
def index
  include_javascript("admin/mootools-1.2-core-yc.js")
  @links = Link.find(:all)
end
```

That will insert a link to admin/mootools-1.2-core-yc.js inside the admin/links/index page. Righto, so now the only thing left to do is get the mootools-1.2-core-yc.js file into the actual public/javascripts/admin directory within the project. What you want to do is alter the vendor/extensions/link_roll/link_roll_extension.rb file so that within the activate method the file is copied over. I'd recommend making a public/javascripts/admin directory within your extension folder, and putting the file in there. Then, when activate is called on the link_roll_extension.rb file, the activate method will copy the file over to the projects public/javascripts/admin directory. You should also delete the file when the deactivate method is called in the link_roll_extension.rb file.

Questions and comments please!
