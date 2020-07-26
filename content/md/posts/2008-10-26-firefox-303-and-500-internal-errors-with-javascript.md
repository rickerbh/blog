{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2008-10-26 11:20:27", :layout :post, :draft? false, :title "Firefox 3.0.3 and 500 Internal Errors with Javascript"}

I had a very confusing situation today with a multipart form that was for uploading a picture to a new web service I'm working on.

In Safari the form upload worked. Even in Internet Explorer 6 the form upload worked (after I fixed the dodgy MIME-type that IE passes through for JPG images - image/pjpeg for those interested).

The form I was trying to submit had the multipart attribute set correctly, and also had some javascript to disable the file selection, form submission button, and show a spinner to indicate that something is happening, and they don't try and submit the file twice if they're sending in a large image.

My submit tag orginally looked like this (Ruby on Rails)

``` ruby

&lt;%= submit_tag 'Upload Photo', :class =&gt; "formbutton", :id =&gt; "submit-button", :onClick =&gt; "$('upload-form').submit();Form.disable('upload-form');Effect.toggle('footnote', 'appear', {duration: 0});Effect.toggle('spinner', 'appear', {duration: 0});" %&gt;

```

I was very confused as it did work in those other browsers, but not in Firefox.

The key to fixing this was to add return false; to the end of the javascript statement...

``` ruby

&lt;%= submit_tag 'Upload Photo', :class =&gt; "formbutton", :id =&gt; "submit-button", :onClick =&gt; "$('upload-form').submit();Form.disable('upload-form');Effect.toggle('footnote', 'appear', {duration: 0});Effect.toggle('spinner', 'appear', {duration: 0});return false;" %&gt;

```

What I find particularly confusing about this is that everything I read on the "return false;" statement leads me to believe that this form should not be submitted - however - return true does not work (500 Internal Server Error returned). But what the hey - it works.
