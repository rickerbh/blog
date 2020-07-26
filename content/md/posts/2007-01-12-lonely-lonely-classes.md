{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2007-01-12 23:19:15", :layout :post, :draft? false, :title "Lonely, lonely classes"}

I just had my most pleasant experience with Singletons ever.  I had a little problem I was trying to solve in ruby, and I contacted a friend of mine who is quite a good developer.  He suggested I should be looking at global variables, to static (class) variables, or singletons.  I gave globals a go, and they didn't give me the love I expected, so I said "bye bye" to them - probably never to be used again.  I next tried Singletons.

Ruby defines a Singleton module.  This is mixin'd with your code thusly:

``` ruby
class Lonely
include Singleton
end
```

Easy peasy.

That, creates a singleton of Lonely for you.  The include modifies the signatures of the new and allocate Class methods (Lonely.new and Lonely.allocate) to be private, and creates (or modifies) some other methods to provide some loveliness (gory details to be found at <a href="http://www.ruby-doc.org/stdlib/libdoc/singleton/rdoc/index.html">http://www.ruby-doc.org/stdlib/libdoc/singleton/rdoc/index.html</a>).

Basically, then all you need to do is call the instance method of the class you have created.

``` ruby
m = Lonely.instance
```

And then you can use m as you like.  You can create more

``` ruby
m = Lonely.instance
n = Lonely.instance
```

And m and n refer to the same instance.  Nice Singleton.  No need for me to check out class variables.  I think my Singleton will be more portable.  BTW - Don't try to .new your class - you'll get a NoMethodError thrown as the new method has been made private by the mixin.

Ruby rocks.

 
