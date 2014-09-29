---
layout: page
title: Security
permalink: /security/
---


This security checklist is a good start to harden your VoIP setup:

* **Principle of minimal privilege:** Try to restrict your setup as much as possible to do exactly what you intended it to do, not more. This principle implicitly applies to all of the following points.

* **Operating System:**

    * Use virtual environments, such as Xen, VirtualBox, OpenVZ, ...
    * Use a chroot environment.
    * Run Yate with a dedicated system user and group.
    * Set ulimits to prevent resource exhaustion.
    * Use application security systems, e.g. AppArmor.
    * Don't run any other server software on the system.
    * Don't let many users access the system.
    * Log admin access.


* **Filesystem:** Restrict access to files:

    * Yate files, scripts and modules should be owned by a different system user than the user that runs Yate. E.g.

            chown -R root:yate /usr/local/etc/yate /usr/local/share/yate

    * Files should be set read-only for the user that runs Yate. E.g.

            chmod -R go-w /usr/local/etc/yate /usr/local/share/yate

    * Files containing passwords or other sensitive information should be set unreadable for others:

            cd /usr/local/etc/yate
            chmod 640 accfile.conf regfile.conf mysqldb.conf

    * Consider using encrypted filesystems to protect sensitive data, e.g. voicemail messages or remote VoIP account credentials.


* **IP Network:**

    * Set up a firewall to restrict access to SIP, rmanager, extmodule, ... and don't forget IPv6.
    * Set up flood protection, e.g. fail2ban.
    * Use a VPN to restrict access to access all or parts of Yate.
    * Configure management services like rmanager and extmodule to listen on localhost only.
    * Configure a dedicated VLAN for VoIP traffic.
    * Protect switch ports with IEEE 802.1x if possible.
    * Set switch ports to be disabled after link is down.


* **Database:**

    * Write your SQL statements with caution: Only use appropriately escaped or whitelisted values in dynamic queries in order to prevent [SQL injection](https://www.owasp.org/index.php/SQL_injection) attacks. Keep in mind, that variables may contain user provided values, such as user agent, caller ID or custom SIP headers.
    * Restrict Yate database user to DELETE, INSERT, SELECT, USAGE, UPDATE. There is no reason for the database to be dropped or altered by a phone call.
    * Think about rejecting suspicious database queries by whitelisting or blacklisting queries before execution using the *regexroute* module.


* **SIP Security:**

    * Only allow SIP methods actually needed, e.g. disable OPTIONS.
    * Don't enable subscribe/notify features to unauthenticated users.
    * Don't leak information about server software versions to the outside. Change the default SIP header *Server:* (or *User-Agent:* for SIP clients) to omit version numbers: `ysipchan.conf`:

            useragent=Foo

    * Filter traffic to other networks, e.g. with a Session Border Controller (SBC).


* **VoIP routing and dialplan considerations:**

    * Avoid routing loops. Yate has an internal loop detection. But bouncing calls from one VoIP server to another and back several times will exhaust resources and provide attackers with a deny-of-service attack surface.
    * Restrict internal numbers to authenticated clients.
    * Categorise clients by source IP, if possible. E.g. internal clients may always have an internal IP.
    * Protect your dialout. Anonymous users or SIP scanners should not be able to generate charges on your telephone bill.
    * Never trust an incoming caller ID. Caller IDs can be faked, is PSTN as well as in VoIP. Also: Obscure caller IDs should be rejected or rewritten at an early routing stage, e.g. allow only digits 0-9, A-D and maybe allow the international `+' character in some cases.
    * Do not allow users to change their caller ID, e.g. set caller ID based on the authenticated username.
    * Explain your dialplan. Draw diagrams. Write tables. Fill Wikis. Anything. Please.
    * Test your configuration. In particular, regular expressions as used to create a dialplan with the *regexroute* module can be tricky.



* **Passwords:**

    * Generate strong and random user passwords, e.g. with [APG](http://www.adel.nursat.kz/apg/).
    * If possible, avoid passwords at all, but use certificates or hardware tokens instead.
    * Protect phone applications, e.g. voicemail, with passcodes longer than four digits. If possible, add additional checks for valid caller-IDs, user authentication credentials, IPs, time of day or other criteria.
    * Users must be able to change their passwords and PINs on their own. They should also be made aware of this feature.



* **Update Strategy:**

    * Regularly check for new versions.
    * Know how to easily update Yate. Take notes on how to compile, deploy, install, upgrade Yate to make life easier for the future you or possibly for other administrators. Also: Store notes where they can be found, e.g. in a file `../YATE_NOTES.txt` or a documentation wiki (or even an offline notebook).



* **Transport Encryption:** Consider setting up encryption if possible:

    * Enable SIP over TLS (SIPS).
    * Enable SRTP.
    * As a client, validate certificates in order to prevent man-in-the-middle attacks.
    * Consider enforcing encrypted calls - SIPS + SRTP - for some numbers, e.g. confidential conference rooms.
    * For performance reasons it may be better to use VPN solutions - e.g. IPSec or OpenVPN - for point-to-point links in some cases.


* **No Logging:**

    * Log nothing unless absolutely required. For personal use, this may be unnecessary. For business use it may even be against privacy laws to store connection data.
    * Think about logging only statistics - e.g. usage counters - without associated names/numbers.
    * A cronjob should be in place to delete old data.


* **Monitoring:** Set up monitoring software in order to know when something went wrong.

* **Security Checks:** Implement as many security features as possible and check them on a regular basis.

* **Disaster Recovery:** Keep your VoIP setup well documented and create automated backups on a regular basis. It should be well known what to do after discovering a security incident - for example:

    * Disconnect from the internet.
    * Restore VoIP setup to a defined state.
    * Find and fix vulnerability, e.g. upgrade software.
    * Change all passwords, PINs, SSH keys, ... and revoke certificates.
    * Inform users.

