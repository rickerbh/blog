---
layout: post
title: "UISearchBar's UITextField"
date: 2012-05-19 10:12
comments: true
categories:
---
For an update that I'm making to [Moving Van](http://click.linksynergy.com/fs-bin/stat?id=*W1h7qYtoaI&offerid=146261&type=3&subid=0&tmpid=1826&RD_PARM1=http%253A%252F%252Fitunes.apple.com%252Fus%252Fapp%252Fmoving-van%252Fid357418069%253Fmt%253D8%2526uo%253D4%2526partnerId%253D30) (_you should buy it now!_) I need to customise the font that is displayed in a UISearchBar's text field. The search bar does not actually expose it's UITextField property, but because the search bar is a UIView, it's trivial to access the field to allow customisation.

    for (UIView *searchSubview in mySearchBar.subviews) {
      if ([searchSubview isKindOfClass:[UITextField class]]) {
        // Do your text field customisation in here
        [(UITextField *)searchSubview setTextColor:[UIColor redColor]];
      }
    }


HTH.