# Get WebID + TLS working for you !

**Introduction**

In this tutorial you will learn how to setup your own certificate and put a public key in your solid pod to be authenticated with your browser. 
Normally you could use a mechanism involving the html <keygen> element such as in [node-solid server](https://github.com/solid/node-solid-server#readme). Unfortunately this is [deprecated by major browsers](https://developer.mozilla.org/fr/docs/Web/HTML/Element/Keygen).

Although, it should be replaced by one of these "one-click" solutions in the future :

* The <keygen> element which is no longer supported could be replaced by an authentication api for browsers ;
* Someone makes an easy to use tool to automatically execute the commands and add the certificate to the OS 😋.  

For now, we will manually create all we need to have a functional WebId certificate.

**Summary**

1. Create an asymetric key pair
2. Create a certificate
3. Import the certificate in your browser
4. Add the certificate to your WebId profile

**Prerequisites**

You must have a [solid server](https://github.com/solid/node-solid-server#readme) running.

## 1. Create an asymetric key pair

You can use the ssh-keygen command. 

**Note** : On Windows you can download the portable version of git and use the binaries in `usr\bin`.

Execute :

    ssh-keygen

This gives you `id_rsa` (private key) and `id_rsa.pub` (public key).

## 2. Create a certificate

### 2.1 Configuration file for openssl

Now we will use openssl. First of all you need a default config file. If you have a local web server with PHP you can copy `path\to\xAMP\bin\php\php7.0.4\extras\ssl\openssl.cnf` for example.

Then you have to append a few lines in order to be compliant with [WebID-TLS](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/tls-respec.html#certificate-example).

It uses the Subject Alternative Name (`SAN`) extension to add extra fields. You have to put your webid in `subjectAltName`.

Append these lines :

    [ SAN ]

    basicConstraints = CA:FALSE
    keyUsage = nonRepudiation, digitalSignature, keyEncipherment, keyAgreement, Certificate Sign 
    subjectAltName = URI:https://test.com/profile/card\#me

**Note** : Don't forget to escape the hashtag (`#`) if you have one in your webid.

### 2.2 Generate the certificate

Now we have to generate a certificate with all the configuration we want including `SAN` extension.

Execute :

    openssl req -x509 -new \
        -config openssl.cnf \
        -days 36500 \
        -key id_rsa \
        -out id_rsa.crt \
        -subj "/C=FR/ST=Gironde/L=Bordeaux/O=MyOrg/CN=Sylvain/emailAddress=sylvain@example.com" \
        -reqexts SAN -extensions SAN

It gives us `id_rsa.crt`.

### 2.3 Output the certificate

Execute :

    openssl x509 -in id_rsa.crt -inform pem -text

You should have something like :

    Certificate:
        Data:
            Version: 3 (0x2)
            Serial Number: 16880934409681490478 (0xea45206815bbd22e)
        Signature Algorithm: sha256WithRSAEncryption
            Issuer: C=FR, ST=Gironde, L=Bordeaux, O=MyOrg, CN=Sylvain/emailAddress=sylvain@example.com
            Validity
                Not Before: Jun  4 18:17:39 2017 GMT
                Not After : May 11 18:17:39 2117 GMT
            Subject: C=FR, ST=Gironde, L=Bordeaux, O=MyOrg, CN=Sylvain/emailAddress=sylvain@example.com
            Subject Public Key Info:
                Public Key Algorithm: rsaEncryption
                    Public-Key: (2048 bit)
                    Modulus:
                        00:d0:c1:c3:0c:87:4f:1f:c1:4f:19:13:ae:29:ba:
                        2b:d1:e9:6c:37:5f:8c:7a:b7:ee:a9:19:97:78:80:
                        d6:2a:66:a5:1d:84:3b:b8:cf:aa:95:95:18:b2:79:
                        32:2f:45:ae:90:79:2e:54:42:08:ea:b3:39:37:5c:
                        92:07:08:01:78:90:43:1d:c1:18:ee:73:9a:b0:be:
                        38:f3:b7:b0:4b:16:06:6d:4a:20:4a:68:27:c5:cc:
                        fa:24:bd:b0:6e:54:62:14:09:2e:02:da:a5:3d:71:
                        ee:c2:e1:0f:e5:fd:36:50:cc:36:0c:b7:9f:26:8e:
                        7d:89:a5:0c:77:f5:4b:fb:09:ad:c3:3b:bf:66:d5:
                        2d:89:df:b3:37:50:c4:00:6b:cb:33:9c:b7:e5:f2:
                        a5:f7:6c:c7:24:ba:e2:97:11:59:ae:22:6c:49:d8:
                        4d:a6:d0:bc:2b:5e:26:5b:a2:55:b3:a2:4d:99:70:
                        a6:d1:22:85:9e:27:6f:5e:35:29:c7:c8:54:4a:b3:
                        c0:2e:16:41:90:76:d8:a2:a7:b9:97:a3:8d:03:7d:
                        69:65:35:2e:f7:f1:f7:50:db:f6:4a:1c:44:c0:e1:
                        19:53:17:06:f6:8c:4b:8b:88:96:ac:cc:c1:0c:fd:
                        55:a7:b9:83:b9:4c:77:b8:95:01:9f:40:1c:86:88:
                        64:83
                    Exponent: 65537 (0x10001)
            X509v3 extensions:
                X509v3 Basic Constraints:
                    CA:FALSE
                X509v3 Key Usage:
                    Digital Signature, Non Repudiation, Key Encipherment, Key Agreement, Certificate Sign
                X509v3 Subject Alternative Name:
                    URI:https://test.com/profile/card#me
        Signature Algorithm: sha256WithRSAEncryption
             08:...:1b
    -----BEGIN CERTIFICATE-----
    ...
    -----END CERTIFICATE-----

*The certificate must contain a `X509v3 Subject Alternative Name` section with your webid in it.*

Copy / paste the data from your console to use later in your WebID profile.

## 3. Import the certificate in your browser

Now we generate a PKCS #12 file to be able to import the certificate in the browser.

Execute :

    openssl pkcs12 -export -in id_rsa.crt -inkey id_rsa -out id_rsa.p12

This gives us `id_rsa.p12`. Import this certificate in your browser.

## 4. Add the certificate to your WebId profile

Now we just have to put the correct information in our WebId profile so the server can authenticate us. In order to be able to do that we must copy the modulus and the exponent of the key.

First be sure your profile has the following namespaces :

    @prefix cert: <http://www.w3.org/ns/auth/cert#> .
    @prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
    @prefix rdfs: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .

Add the following line in the <#me> subject :

    cert:key <#public_key> .

Add a new subject to the profile :

    <#public_key> 
        a cert:RSAPublicKey;
        rdfs:label "made on 2017-06-04 on my laptop";
        cert:modulus "d0c1c30c874f1fc14f1913ae29ba2bd1e96c375f8c7ab7eea919977880d62a66a51d843bb8cfaa959518b279322f45ae90792e544208eab339375c920708017890431dc118ee739ab0be38f3b7b04b16066d4a204a6827c5ccfa24bdb06e546214092e02daa53d71eec2e10fe5fd3650cc360cb79f268e7d89a50c77f54bfb09adc33bbf66d52d89dfb33750c4006bcb339cb7e5f2a5f76cc724bae2971159ae226c49d84da6d0bc2b5e265ba255b3a24d9970a6d122859e276f5e3529c7c8544ab3c02e16419076d8a2a7b997a38d037d6965352ef7f1f750dbf64a1c44c0e119531706f68c4b8b8896acccc10cfd55a7b983b94c77b895019f401c86886483"^^xsd:hexBinary;
        cert:exponent 65537 .

**Note** : You have to make some modifications to the modulus :

* delete all `:` between bits ;
* delete all spaces and line break ;
* delete the two leading zeros.

You should have something like :

    @prefix cert: <http://www.w3.org/ns/auth/cert#> .
    @prefix xsd: <http://www.w3.org/2001/XMLSchema#> .
    @prefix rdfs: <http://www.w3.org/1999/02/22-rdf-syntax-ns#> .
    @prefix solid: <http://www.w3.org/ns/solid/terms#>.
    @prefix foaf: <http://xmlns.com/foaf/0.1/>.
    @prefix pim: <http://www.w3.org/ns/pim/space#>.
    @prefix schema: <http://schema.org/>.
    @prefix ldp: <http://www.w3.org/ns/ldp#>.

    <>
        a foaf:PersonalProfileDocument ;
        foaf:maker <#me> ;
        foaf:primaryTopic <#me> .

    <#me>
        a foaf:Person ;
        a schema:Person ;

        foaf:name "sylvain@example.com" ;
        
        foaf:knows <http://csarven.ca/#i> ;

        solid:account </> ;  # link to the account uri
        pim:storage </> ;    # root storage

        solid:inbox </inbox/> ;
        ldp:inbox </inbox/> ;

        pim:preferencesFile </settings/prefs.ttl> ;  # private settings/preferences
        solid:publicTypeIndex </settings/publicTypeIndex.ttl> ;
        solid:privateTypeIndex </settings/privateTypeIndex.ttl> ;
        cert:key <#public_key> .

    <#public_key> 
        a cert:RSAPublicKey;
        rdfs:label "made on 2017-06-04 on my laptop";
        cert:modulus "d0c1c30c874f1fc14f1913ae29ba2bd1e96c375f8c7ab7eea919977880d62a66a51d843bb8cfaa959518b279322f45ae90792e544208eab339375c920708017890431dc118ee739ab0be38f3b7b04b16066d4a204a6827c5ccfa24bdb06e546214092e02daa53d71eec2e10fe5fd3650cc360cb79f268e7d89a50c77f54bfb09adc33bbf66d52d89dfb33750c4006bcb339cb7e5f2a5f76cc724bae2971159ae226c49d84da6d0bc2b5e265ba255b3a24d9970a6d122859e276f5e3529c7c8544ab3c02e16419076d8a2a7b997a38d037d6965352ef7f1f750dbf64a1c44c0e119531706f68c4b8b8896acccc10cfd55a7b983b94c77b895019f401c86886483"^^xsd:hexBinary;
        cert:exponent 65537 .

Now you only have to load a url of your solid pod. The browser should ask you to select a key. Select the one you've just added to the browser. You're authenticated !

# Resources 

[Old Tuto](https://www.w3.org/wiki/index.php?title=Foaf%2Bssl/HOWTO#How_to_create_a_certificate_which_includes_the_WebID_URI_for_multi-purpose_use)
