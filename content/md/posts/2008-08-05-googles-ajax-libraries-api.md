{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2008-08-05 15:01:12", :layout :post, :draft? false, :title "Google's AJAX Libraries API"}

This might be old news (released end of May, I'm soooo behind the times), but Google are supporting hosting for popular javascript libraries, and they are promoting people using their copies of the libraries rather than hosting their own.  I only just found this out, and I think it's pretty cool

<a href="http://code.google.com/apis/ajaxlibs/" target="_blank">http://code.google.com/apis/ajaxlibs/</a>

The service supports either <a href="http://code.google.com/apis/ajax/documentation/" target="_blank">programmatic inclusion</a> using the google jsapi library, or <a href="http://code.google.com/apis/ajaxlibs/documentation/index.html#AjaxLibraries" target="_blank">linking directly</a> to their hosted copies.

Good things about this
<ul>
	<li>You save bandwidth costs, as your users download the libraries from google</li>
	<li>Your users get faster browsing ON OTHER DOMAINS, because the more people that use a single source, the more caching should happen</li>
</ul>
Bad things about this
<ul>
	<li>Google have more opportunity to <span style="text-decoration: line-through;">don't</span> be evil - the biggest concern for me is their increased ability to understanding which users use your application, and propensity for understanding <em>how</em> they use it.  A malicious individual within the company could put some nasty code in the js to screw with your site, or monitor and transmit back to googlebase what the user is up to.  It's <em>very</em> unlikely, but it could happen.</li>
</ul>
