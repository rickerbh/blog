{:author "<a href=\"/about\">Hamish Rickerby</a>", :date "2008-05-31 16:34:49", :layout :post, :draft? false, :title "Mnemonic Password Generator - a la Ruby"}

I wanted to have a mnenonic password generator for a little application that I'm writing in ruby, and couldn't find one out there.  So, I rolled my own.  Actually, it's a knock-off of a <a href="http://homework.nwsnet.de/">Python one I found</a>, but that doesn't really matter.

Do with this what you will.  Hope it's useful to someone.

``` ruby
def generateMnemonicPassword(letters=8, digits=4)
consonants = "bdfghklmnprstvwz"
vowels = "aeiou"
password = ""
(1 .. letters).each do |i|
range = i%2 == 1 ? consonants : vowels
password = password + range[rand(range.length), 1]
end
(1 .. digits).each do |i|
password = password + rand(10).to_s
end
password
end
```

It outputs fun and pronounceable passwords such as tepelopu8058, vonobuba6145 and kipowetu0270.

Enjoy!
