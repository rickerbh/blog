---
author: <a href="/about">Hamish Rickerby</a>
comments: yes
date: '2009-02-26 19:42:04'
layout: post
slug: dealing-with-semantichacker-in-ruby
status: publish
title: Dealing with SemanticHacker in Ruby
wordpress_id: '292'
? ''
: - ruby
---

I'm having a bit of a play with <a href="http://www.semantichacker.com/">SemanticHacker</a> at the moment, seeing what their service thinks of some text I'm poking at it.  To make this a bit simpler I created a small ruby lib that wraps their API to make my life a little easier.  And hopefully it'll make someone elses life easier - enjoy.

(It requires <a href="http://github.com/why/hpricot/tree/master">hpricot</a> for XML parsing - make sure it's installed)

``` ruby
require 'rubygems'
require 'hpricot'
require 'cgi'
require 'open-uri'

class SemanticHacker

  URL = "http://api.semantichacker.com"
  attr_accessor :token, :doc, :content

  def initialize(token)
    @token = token
  end

  def get_signature(content)
    @content = ::CGI::escape(content)
    url = "#{URL}/#{@token}/signature?content=#{@content}"
    @doc = Hpricot.XML(open(url))
  end

  def get_concepts(content)
    @content = ::CGI::escape(content)
    url = "#{URL}/#{@token}/concept?content=#{@content}"
    @doc = Hpricot.XML(open(url))
  end

  def get_categories(content)
    @content = ::CGI::escape(content)
    url = "#{URL}/#{@token}/category?content=#{@content}"
    @doc = Hpricot.XML(open(url))
  end

  def type
    (doc/:response/:about/:systemType).inner_html
  end

  def config_id
    (doc/:response/:about/:configId).inner_html
  end

  def categories
    response = []
    (doc/:response/:categorizer/:categorizerResponse/:categories/:category).each do |item|
      response << {:id => item.attributes['id'], :weight => item.attributes['weight']}
    end
    response
  end

  def concepts
    response = []
    (doc/:response/:conceptExtractor/:conceptExtractorResponse/:concepts/:concept).each do |item|
      response << {:label => item.attributes['label'], :weight => item.attributes['weight']}
    end
    response
  end

  def signatures
    response = []
    (doc/:response/:siggen/:siggenResponse/:signature/:dimension).each do |item|
      response << {:index => item.attributes['index'], :weight => item.attributes['weight']}
    end
    response
  end

end
```

And to make things happen

``` ruby
sh = SemanticHacker.new("mysecrettoken")
sh.get_signature("Wow!  Some semantic analysis on my text")
puts sh.signatures.inspect 
#returns an array of hashes with the weights and indexes of the categories
``` 