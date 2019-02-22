# SSHD Kerberitzat
## @edt ASIX M11-SAD Curs 2018-2019

**edtasixm11/k18:sshd** Servidor SSHD *kerberitzat*. Servidor ssh que permet
  l'accés d'usuaris locals i usuaris locals amb autenticació kerberos. El
  servidor s'ha de dir sshd.edt.org.

Per fer que el servidor sshd sigui *kerberos aware* cal:

 * instal·lar.hi el paquet krb5-workstation.

 * copiar el fitxerd e configuració kerberos client */etc/krb5.conf*.

 * crear un principal al servidor kerberos coresponent al host sshd. En aquest cas concret es crea
  una entrada *host/sshd.edt.org*.

 * propagar aquest principal (de host)  al keytab del servidor sshd. Aquí s'utilitza un mecanisme 
  *pèssim*, des del client, desatesament, es connecta amb usuari i passwd (text pla) via *kadmin* al
  servidor kerberos i exporta les claus del principal *host/sshd.edt.org* via *ktadd*.

 * configurar el servidor sshd per permetre l'autenticació kerberos, cal modificar el fitxer de 
 configuració del servei */etc/ssh/sshd_config*:

```
# Kerberos options
KerberosAuthentication yes
KerberosTicketCleanup yes
```

**Atenció:**

Si el principal de host que s'ha creat al servidor kerberos és *host/sshd.edt.org* es podrà realitzar
l'accés kerberitzat **només** si es connecta al servidor usant aquest hosname. És a dir, amb les ordres:
```
ssh user01@sshd.edt.org  (OK)
ssh user01@localhost     (NO!)
```
La primera d'elles peermetrà l'accés kerberitzat, la segona no.


Execució:
```
docker run --rm --name sshd.edt.org    -h sshd.edt.org    --net mynet -d edtasixm11/k18:sshd
```

### Accés kerberitzat / Accés normal

Feu atenció al significat d'accés kerberitzat! Si l'usuari user01 és un usuari vàlid en el host sshd.edt.org
(provingi el seu information privider de /etc/passwd o de ldap) i la seva autenticació és kerberos (el seu 
password està desat al kerberos), en un accés kerberitzat, si l'usuari ja disposa de ticket podrà accedir
al serveidor sshd sense cap password. **atenció** usualment quan això no va és causat per el keytab, no s'ha exportat
bé, no s'ha creat bé, o no coincideixen els noms assignats al host.

Exemple-1

Des del propi container l'usuari local01, sense estar en posessió de cap ticket, realitza l'ordre *ssh user01@sshd.edt.org*,
el servidor li demanarà el password, en tractar-se d'un usuari de *AP* kerberos, verificarà l'autebnticació contra el
servidor kerberos (el *IP* l'ha obtingut de /etc/passwd).

Exemple-2

Des del propi container l'usuari local01 sol·licita un ticket de user02 amb l'ordre *kinit user02*. Si s'autentica 
correctament amb el password de kerberos obté un tiket. Seguidament l'usuari local01 realitza l'ordre *ssh user01@sshd.edt.org*
i conectra automàticament al servidor ssh sense que se li demnai el password.

Perquè? perquè està jà en possesió d'un ticket kerberos vàlid que el servidor sshd verifica i li permet iniciar sessió
ssh sense necessitat de demanar-li el password (similar a l'accés per clau pública).


