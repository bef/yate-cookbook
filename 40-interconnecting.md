---
layout: page
title: Interconnecting
permalink: /interconnecting/
---

## Connecting LCR

```
-------------           -------------
|    Yate   |-----------|     LCR   |
|172.16.40.2|   (SIP)   |172.16.40.2|
-------------           -------------
```

Running Yate and [LCR (Linux Call Router)](http://www.linux-call-router.de) on the same host - e.g. localhost - is possible. However if the setup is more complicated - e.g. Yate + Asterisk + LCR or several Yates and LCR or several LCRs - you can save a lot of time and effort by separating each VoIP server into its own (virtual) environment with its own network interface. Just keep in mind, that real hardware access may require PCI passthru or equivalent for better timing.

`LCR interface.conf`:

    [Yate_SIP]
    #sip <local ip>[:<local port>] <remote ip>[:<remote port>]
    sip 172.16.40.1:5061 172.16.40.2
    earlyb no
    tones yes

If not already covered by the a listener define another listener in `Yate ysipchan.conf`:

    [listener lcr]
    enable=yes
    addr=172.16.40.2
    port=5060


Routing from LCR to Yate is shown by the following example. Calling *01980123* will call *123* via interface Yate_SIP. Calling *01999123* will call *01999123* as the `goto` does not consume matched digits.
`LCR routing.conf`:

    [main]
    dialing=01980	: extern interfaces=Yate_SIP
    dialing=01999	: goto ruleset=to_yate

    [to_yate]
    	: extern interfaces=Yate_SIP


Routing from Yate to LCR is done the usual way - assuming 0... will be routed to LCR:
`Yate regexroute.conf`:

    ^\(0.*\)$=sip/sip:\1@172.16.40.1:5061


*Note*: Try to avoid routing loops.


## Connecting Asterisk (Scenario 1)

N-to-N trunking without registration or authentication via SIP protocol - (assuming VPN or other trust relationship between hosts and static IPs):

```
--------           ------------
| Yate |----SIP----| Asterisk |
--------           ------------
```

`Asterisk sip.conf` for incoming calls:

```INI
[world]
type=peer
host=office.example.com
context=default
permit=192.168.88.99/255.255.255.255
```

Calling from Yate to Asterisk: - assuming a two-digit extension: `Yate regexroute.conf`

```INI
^..$=sip/sip:\0@192.168.88.98
```

Calling from Asterisk to Yate: `Asterisk extensions.ael`

```
context internal {
	_X. => {
		Dial(SIP/${EXTEN}@192.168.88.99);
		Hangup;
	};
};
```

## Connecting Asterisk (Scenario 2)

Yate registers to Asterisk via IAX2 protocol.

```
--------    --------           ------------    ---------
| PSTN |<---| Yate |--(IAX2)-->| Asterisk |--->| Phones|
--------    --------           ------------    ---------
```

This is one practical solution for N-to-N trunking with registration and two way password authentication. The opposite direction where Asterisk registers to Yate would be more straight forward, however routing from Yate to Asterisk can become rather complicated.

`Asterisk sip.conf`:

```INI
[world]
type=friend
;host=office.example.com
host=dynamic
secret=SECRET_PASSWORD_HERE
context=default
permit=192.168.88.99/255.255.255.255
```

`Yate accfile.conf` for registration and outgoing connections:

```INI
[office]
enabled=yes
protocol=iax
username=world
password=SECRET_PASSWORD_HERE
server=192.168.88.98
interval=30
```

`Yate regfile.conf` for incoming connections:

```INI
[x1]
password=SECRET_PASSWORD_HERE
```

Calling from Yate to Asterisk - assuming a two-digit extension: `Yate regexroute.conf`:

```INI
^..$=line/\0;line=office
```

Calling from Asterisk to Yate: `Asterisk extensions.ael`:

```
context internal {
	_X. => {
		Dial(IAX2/x1:SECRET_PASSWORD_HERE@192.168.88.99/${EXTEN});
		Hangup;
	};
};
```

