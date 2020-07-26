{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2005-12-29 22:35:00", :layout :post, :draft? false, :title "select with value - rubyonrails"}

Grrr, just spent a little while playing with the "select" form tag in rails.  Pain in the ass.  The issue I was having is that I want to give users an option of 2 values with a default selected value on the "new" form.  I also have a edit form that I want to let users update their objects with, and again, only 2 values.  These values are static, and I didn't want to have the pain of creating a db table, and an object to represent it.  That would result in unnecessary code, and unnecessary database hits.

The issue with the "select" tag is that I cannot determine how to set the selected value, without using a collection of objects.  I had a look for a solution for this problem, and found <a href="http://wiki.rubyonrails.com/rails/pages/HowtoUseFormOptionHelpers">this site</a>.  This gave me a helper that lets me put in an object, with some methods to get the value and text options for my option set.  that was alright.  But, I didn't want the full object model to be supported.  I was after a very simple implementation with using a hash, and a string as the selected value.

I hacked around with the code, and this is what I came up with.  Please feel free to pillage this.


def hash_select_with_current(object, method, hash, current_value)
  result = "<select name=\"#{object}[#{method}]\">"
  hash.each do |key, value|
    if current_value == value
      result << "<option selected=\"selected\" value=\"#{value}\">#{key}</option>"
    else
      result << "<option value=\"#{value}\">#{key}</option>"
    end
  end
  result << "</select>"
  return result
end
```

Hope that's useful for someone.
