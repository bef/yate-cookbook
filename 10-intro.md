---
layout: page
title: Introduction
permalink: /
---

## About
This document is a collection of practical solutions and ideas - *recipes* - related to the [Yate telephony engine](http://yate.null.ro/). Recipes give specific solutions to specific problems encountered in real life. Some recipes were derived from practical experience during the 29th and 30th Chaos Communication Congress in Hamburg, Germany. Other solutions were encountered while setting up and maintaining our public VoIP provider [EPVPN](https://eventphone.de/epvpn) (Eventphone Virtual Phone Network).

The technical minded person may find a jump start into Yate or use it as a reference guide.
Personally I prefer to write down experiences before they slip my mind and may be lost forever.
Please feel free to share this document (under the terms of this document's license). Information wants to be free.

## The Author

Ben Fuhrmannek is a computer scientist with a background in software development and IT security. In his spare time, among other things, he likes to indulge in open communication.

For more information check out my list of [presentations and papers](http://www.fuhrmannek.de/pnp/).

## Alternatives to Yate

The eventphone team has been evaluating VoIP software since 2004. And we came across quite a few working solutions for our use case: Provide secure, stable and reliable VoIP connectivity for more than 1000 hackers during conferences. This is my personal opinion:

* **PBX4Linux, now LCR:** LCR comes with the idea of an ISDN PBX, now with SIP connectivity. We are using LCR as our central routing mechanism to interconnect DECT, Yate, Asterisk, GSM and Dialin/Dialout.
* **Asterisk (and many forks):** This PBX opened my eyes in regard to programming VoIP applications. AGI and AEL are quite powerful interfaces and very intuitively programmed. Complex IVRs, games, automated calls and many more features make this PBX software almost perfect for every use case. However, there has not been a single hacker event without the need for a *while-true* loop to restart asterisk every hour or so. A similar setup with Yate lasted the whole 29c3 and 30c3 without a single unscheduled restart. The people of the rather popular [Gemeinschaft PBX](http://amooma.de/gemeinschaft) seem to come to a similar [conclusion](http://www.freeswitch.org/node/373) and switched from Asterisk to FreeSWITCH: "FreeSWITCH is better, more stable, more secure and gives higher performance."
* **FreeSWITCH:** This PBX caught my attention because it took me about 10 minutes form download to the first call. Configuration files are XML, there are many (maybe even a lot of) modules available and the PBX can be scripted in every popular scripting language. Hardware support seemed a bit limited at the time - in 2005 I found only Sangoma ISDN card drivers. An alpha version of mISDN worked just fine when I tried it. Just like VIM vs. Emacs, RISC vs. CISC, for me this is a tie between FreeSWITH and Yate.
* **OpenSER / Kamaillo / OpenSIPS:** These *Open Source SIP Servers* seem to focus on SIP and do their job well.
* **YXA:** As a friend of software written in Erlang, particularly PBX software, this SIP router could have been a good successor for Asterisk. However after a day or so of configuration efforts I chose to focus on alternatives.


## License

This work is licensed under a [Creative Commons Attribution 4.0 International License](http://creativecommons.org/licenses/by/4.0/).

You are free to:

* **Share** — copy and redistribute the material in any medium or format
* **Adapt** — remix, transform, and build upon the material

for any purpose, even commercially.

Under the following terms:

* **Attribution** — You must give appropriate credit, provide a link to the license, and indicate if changes were made. You may do so in any reasonable manner, but not in any way that suggests the licensor endorses you or your use.

**No additional restrictions** — You may not apply legal terms or technological measures that legally restrict others from doing anything the license permits.

## Referencing Guideline

When quoting or referencing this document, it is good style to mention

* **the author's name:** Ben Fuhrmannek - *Note: Derivative and contributed work may have a different author or several authors as indicated on the page or chapter.* The github username - here @bef - is sufficient when referencing on github.
* **document title:** {{ site.title }}
* **document version, revision or date**: see [https://github.com/{{ site.github_project }}](https://github.com/{{ site.github_project }}) for the latest change. If unsure, use the access date.
* **original URL:** https://bef.github.io{{ site.baseurl }}

*Note:* Be aware, that the document, URLs and the document structure may change over time. It's best not to share a specific URL only, but rather a descriptive reference including chapter name.

### Examples:

* Example reference in print:

    Fuhrmannek, B. (2014-10-01) {{ site.title }}. (Internet). Available from {{ site.projecturl }}. Accessed 29.06.2048.

* Formal web reference:

        <a href="{{ site.projecturl }}">{{ site.title }}</a> by Ben Fuhrmannek

* Casual web reference (specific chapter) when referencing from github:

        See <a href="{{ site.projecturl }}/security/">{{ site.title }} / Security</a>.

### Bad Examples:

* See here. *(no title)*
* Look what I did: ... *(invalid attribution)*
* Ad. Ad. (this work)... Ad. Ad. *(bad context: The document has been released with a commercial use license. But using my name or work in connection with link spam sites is just wrong.)*

