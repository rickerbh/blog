---
layout: post
title: "Medical Regulation and Apps"
date: 2013-03-22 10:23
comments: true
categories:
---
Yesterday I read an [article in Venture Beat](http://venturebeat.com/2013/03/19/health-app-makers-to-feds-dithering-on-regulation-is-stifling-innovation/) covering part of a series of hearings US Congress is holding on regulation of health and medical apps on smartphone and tablet devices. My _significantly_ paraphrased version of the article is that the FDA are unsure if and how health apps should be regulated, and that representatives from the "app" industry are pushing for no regulation for consumer facing apps.

I have a slightly different take on this. To me, the question is simple (and what is driving health authorities globally to address this issue): can health apps be considered to be medical devices?

Medical devices are subject to legislation worldwide.  In fact, often requiring prior market approval through mechanisms similar to which pharmaceutical companies must obtain approval for medicines. Clear guidance exists on the regulation which surrounds medical devices, and this includes the legislative definition of what constitutes a medical device.  As a snapshot (and these definitions can be considered consistent with legislation in many other countries too), the definition of a medical device in Australia, the EU, and US clearly includes apps which diagnose, prevent, monitor, treat, alleviate disease.

{% codeblock Australian Therapeutic Goods Act 1989 http://www.comlaw.gov.au/Series/C2004A03952 %}
41BD A medical device is:
  a. any instrument, apparatus, appliance, material or other article (whether used alone or in combination, and including the software necessary for its proper application) intended, by the person under whose name it is or is to be supplied, to be used for human beings for the purpose of one or more of the following:
    i. diagnosis, prevention, monitoring, treatment or alleviation of disease;
    ii. diagnosis, monitoring, treatment, alleviation of or compensation for an injury or handicap;
    iii. investigation, replacement or modification of the anatomy or of a physiological process;
    iv. control of conception;
  and that does not achieve its principal intended action in or on the human body by pharmacological, immunological or metabolic means, but that may be assisted in its function by such means; or
  b. an accessory to such an instrument, apparatus, appliance, material or other article.
{% endcodeblock %}

{% codeblock EU Directive 2007/47/ec (paraphrased) %}
Any instrument, apparatus, appliance, software, material or other article that is used alone or in combination, including software specifically for diagnostic or therapeutic purposes, that the manufacturer intends for use in human beings. Such devices are used for:
 - Diagnosis, prevention, monitoring, treatment, or alleviation of disease
 - Diagnosis, monitoring, treatment, alleviation of, or compensation for an injury or handicap
 - Investigation, replacement, or modification of the anatomy or of a physiological process
 - Control of conception
{% endcodeblock %}

{% codeblock US Food, Drug and Cosmetic Act Section 201(h) %}
Medical machine, contrivance, implant, in vitro reagent, or other similar or related article, including a component part, or accessory that is:
 - Recognized in the official National Formulary, or the United States Pharmacopoeia, or any supplement to them
 - Intended for use in the diagnosis of disease or other conditions, or in the cure, mitigation, treatment or prevention of disease, in man or other animals
 - Intended to affect the structure or any function of the body of man or other animals, and does not achieve any of its primary intended purposes through chemical action within or on the body of man or other animals and does not depend on metabolic action to achieve its primary intended purposes.
{% endcodeblock %}

The key, and common, part to all of these is the use of the device in _diagnosis, monitoring, and treatment of a disease_.

Regardless of the user of the device, regulation exists. Medical devices such as syringes, contact lenses, condoms, and bandages are all available to the public, and regulated (at least in Australia). If the same rule that is being proposed for apps was applied for these devices, anyone could make devices such as bandages, and the public would have no guarantee as to the quality, safety, and efficacy of these devices.

Devices and medicines are regulated to protect patients (and the general public) from faulty or harmful devices and drugs, and misdiagnosis or mistreatment from inaccurate information being provided to medical professionals who need to make decisions on the basis of that information. The consequences for patients from medical misdiagnosis from a medical app can be a lot more severe than (lets say) the physics calculations not accurately representing gravity in Angry Birds. Death or permanent disability is a real possibility with inaccurate (or just plain wrong) information being captured by medical apps and used for diagnosis.

Getting a medical device approved can be a very costly exercise. Typically, trials with verifiable data are required to show accuracy and stability in the product. There is also the paperwork required to submit the device to the health regulator in a country, and this submission process needs to be repeated for each health regulator in each country you want to sell your device in.{% fn_ref 1 %} However, due to the high cost (and high barrier to entry{% fn_ref 2 %} in the marketplace), if you can jump through the regulatory hurdles, you may find a relatively competition-free market. That's the reward for navigating the processes.

There is also a comment in that article that "Developers are mystified by the rules in this highly regulated industry". If this is the case, maybe developers need to do the same as other players in the health industry and either employ, or contract in, medical regulatory professionals. Mystification is no excuse for not playing by the rules.

_Ignorantia juris non excusat_.

Need help?
----------

[happtic](http://happtic.com) (my company) provides consulting services related to health regulations for medical apps. If you're looking for help determining if an app would be considered a medical device, help with understanding regulatory processes, or help with regulatory submissions please [get in touch](mailto:contact@happtic.com).

{% footnotes %}
  {% fn %} Unless you're in the EU. There is a single health agency for Europe.
  {% fn %} Regulatory requirements are only one significant barrier to entry in certain industries. Other industries (such as telecommunications and energy suppliers) also have a large infrastructure requirement requiring both capital investment and significant time. Medical Apps don't typically have this barrier.
{% endfootnotes%}