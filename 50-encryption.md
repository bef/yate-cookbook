---
layout: page
title: Encryption
permalink: /encryption/
---

## TLS Encryption (SIPS/SRTP Server)


These few steps will enable TLS for SIP and RTP in Yate:

1. Get or create a certificate, e.g. create a self-signed certificate:

        openssl genrsa -out key.pem 2048
        openssl req -new -key key.pem -out request.pem
        openssl x509 -req -days 3650 -in request.pem -signkey key.pem -out certificate.pem
        chmod 400 *.pem

    key.pem and certificate.pem can be moved to /usr/local/etc/yate/ssl. 

2. Configure the openssl module: `openssl.conf`

    ```INI
    [general]

    [server_context]
    enable=yes
    domains=voip.example.com
    certificate=ssl/certificate.pem
    key=ssl/key.pem
    ```

    And enable openssl.yate in yate.conf (set openssl.yate=true in section [modules]). The created [server_context] can be used by several modules: jabberserver, ysipchan, rmanager

3. Configure SIP module: `ysipchan.conf`

    ```INI
    [general]
    ;...
    secure=enable
    ;...
    [listener ssl]
    enable=yes
    type=tls
    ;addr=172.16.88.12
    port=5061
    sslcontext=server_context
    ```

4. Configure all your clients to accept the generated certificate as valid.


Optional steps:

* DNS configuration:
    Set NAPTR and SRV records. [NAPTR](https://en.wikipedia.org/wiki/NAPTR_record) (Name Authority Pointer) records provide a way to point to the correct VoIP resource and may even contain a regular expression to rewrite the request.
    [SRV](https://en.wikipedia.org/wiki/SRV_record) records point services to hostnames and ports.

        example.com. IN NAPTR 10 0 "s" "SIPS+D2T" "" _sips._tcp.example.com.
        example.com. IN NAPTR 20 0 "s" "SIP+D2T" "" _sip._tcp.example.com.
        example.com. IN NAPTR 30 0 "s" "SIP+D2U" "" _sip._udp.example.com.
        _sips._tcp.example.com. IN SRV 10 0 5061 voip.example.com.
        _sip._udp.example.com. IN SRV 10 0 5060 voip.example.com.
        _sip._tcp.example.com. IN SRV 10 0 5060 voip.example.com.

    This example contains a complete set of entries for SIPS, SIP over UDP and SIP over TCP. If you have only enabled listeners for one or two of these services, then don't set the others.

* Encourage the use of encryption by blocking some (or all) numbers for unencrypted connections: `regexroute.conf`:

    ```INI
    ^8044$=if ${ip_transport}^TLS$=if ${encryption}.=if ${crypto}.=conf/tlsonly;maxusers=23;lonely=true
    ```

    This will allow participants to enter a conference room (`conf/tlsonly`) when the signaling connection is TLS encrypted (e.g. SIPS) and also data encryption is offered. *Note*: At this point, the client may still negotiate to use an unencrypted data channel or renegotiate the data channel later.

* Store `oconnection_id` in user database:
    In order to call clients connected by TLS (or TCP), Yate needs to be able to find the TCP connection associated with the client. This is done by looking up (and setting) the channel variable `oconnection_id`. (Actually, the channel variable `route_params` contains a list of variables to be looked up for routing, but for this use case it's just `oconnection_id`.) The file backend (regfile module) can do this automatically. Database backends must store this field explicitly.

