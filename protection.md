# Protection of the data

The Passport Controller is a data broker that provides an intercept point to work in cooperation with the Trust Authority to transform raw data into Trusted Data Objects. This is the data protection activity.

A **Trusted Data Object** (or simply **TDO**) protects the data at the point of exfiltration. That way, data outside the security perimeter is still secured. To do so, IBM DPP creates a copy of a data source, and save it as a TDO. Data and the security mechanism makes the TDOs. The security of the data stays with the data. 

Can be named a TDOs:
* the whole protected table
* a protected table extracts(1-n rows)
* or protected table elements (1-n cells)

As a reminder, and to prove that a TDO secures data, you can find below a TDO sample.
```
 ##P2{
    #K:demompl.mykeyforapp1;;,
    #I:"/orig_cif_number";,
    #D:1U96v8LRGFcv4sg.Kkmg0kxT7yVv2ctPLk331w.fVp5gRbYSlysy0g3gW4n+Q;Long
    }
```

There are two main data protection use cases:
* **Protection** – In this case the Passport Controller protects the data (according to the policy) and stores the protected data (TDOs) into the target DBMS. Here there is a single copy of data saved as TDOs.
* **Protect and then enforce** – In this case, the Passport Controller will be established as a proxy for accessing the protected table and will intercept the SQL requests and apply enforcement to the data before it is returned to the consumer. This is using a single copy of the data to provide multiple views.

Let's see in action both use cases.

## 1. Data Protection with IBM Data Privacy Passports

### 1.1 Protection of the data
:white_check_mark: **Objective:** Data is protected at the exfiltration point and off the platform.

You can find below how an IBM Data Privacy Passports administrator creates a TDO, from a particular DBMS, to a target DBMS (source may be the target). I took as example a source DBMS being a Linux on IBM Z PostgreSQL, and a target DBMS being AWS Oracle.
* **(1a)** DPP Admin connects to DPP and uses DPP SQL functionalities.
* **(1b)** DPP SQL query the source DBMS according to the DPP Admin need.
* **(1c)** Source DBMS sends back SQL query output to DPP as it is (in the clear).
* **(1d)** Sent back data is stored at first in a temporary local view.
* **(1e)** The Passport Controller apply then data protection policy to the data, and saved it as a table (this is the TDO) on target DBMS.

You can find below how a JDBC application experiences an SQL Query directly to a TDO.
* **(2a)** User connect to an URL pointing to a DBMS hosting a TDO, and SQL query it. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ...
* **(2b)** DBMS checks provided credentials, and send back SQL query output to user (protected data). The data is still protected. The only way to experience the data in the clear is to send back the TDO to the Passport Controller.

**Note:** IBM DPP will must use a valid credentials on target DBMS in order to be able to create the TDO table.

![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protection.png?raw=true)

### 1.2 Direct SQL query to an existing TDO

:exclamation: Data being protected for exfiltration, the data is secured. Instructions to decrypt the data is provided in the metadata. It will help Passport Controller to find-out the appropriate and required keys from key-stores.

:exclamation: By the policy it is possible to let some column into the clear.

:computer: Let's experience what data looks like in a TDO:

```
beeline -u 'jdbc:postgresql://ec2-35-180-97-38.eu-west-3.compute.amazonaws.com/userdb' -n myuser -p myuser -d org.postgresql.Driver -e 'select * from protected_customer limit 10;';
```
Expected output is:
```
Connecting to jdbc:postgresql://ec2-35-180-97-38.eu-west-3.compute.amazonaws.com/userdb
Connected to: PostgreSQL (version 10.10 (Ubuntu 10.10-0ubuntu0.18.04.1))
Driver: PostgreSQL JDBC Driver (version 42.2.11)
Transaction isolation: TRANSACTION_REPEATABLE_READ
                                                       orig_cif_number                                                        |                                                    gender                                                    |                                                       first_name                                                        |                                                      last_name                                                      |                                                 age                                                  |                                                      sin                                                       |                                                                 email                                                                  |                                                            phone                                                             |                                                                              mailing_address                                                                               |                                                 prov_abbr                                                  |                                                    postal_code                                                     
------------------------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------------------------------------------------------------+---------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+------------------------------------------------------------------------------------------------------------+--------------------------------------------------------------------------------------------------------------------
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1U96v8LRGFcv4sg.Kkmg0kxT7yVv2ctPLk331w.fVp5gRbYSlysy0g3gW4n+Q;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1/k3sK2+9.8a3lAw6V1FZ+Zz6kMQ8TYg.WsnjBNdpqudyvPqX/albUg;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1vFoPW5VBuJuTh5RD/w.YAIB80N6WC3cgnjdVga+ZA.cLwYRQyh0pOzhGJfAG6rqA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1d9qy2CTeCsMF.m9ar0qKkweqawdGpLi9aSg.x1vqUqraLF8OX/Tgjpf2Mg;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:14As.G73dkXBI1P7syAHtSPU02w.FW4sXvyxOpoZ6RBl7alu6Q;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:11wCyJlGcK74b.9TqCraasDjK7USOMQpyvIw.fjU6jfykqDql4q5iJLevIg;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:145jiFVOrIS3o4FSDYwyCLNfhIoq2a2w.MZGJ4gP2xo+AyLG4q/baiA.hdXM0pCBL0CwFhALI8HT4A;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:14TaB3Tv/Nt8sUAz2K6U.BId+BDG9zUhGbmeYMo+3rw.ytIArjkAnFjVNmLiOBPJAQ;}          | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1R1sFIH45nWXcUQIv0S9Ox4LL8A0ttaaTRvyqgnxD.qeQJmlke49a6rsK8ah53vw.Wmth3+kT0anzNtdQQWdLvg;}                         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1cLgfpg.Pw2m+XS8up3OugrvK5KJFA.20RoyMRIiRNoYc8aEZt9wA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1H+HPXmT3g/5o.au/gQOBkbniFYIIDUGuNjg.NRN0lUlfkL2gkxKTJPgjTg;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1xazXUJ+a/mXefg.6jSQZq1/60I9AcOdY87oow.zxLDjcwloUqZ1sYYl15Arg;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1hHdk2S7Xh6k.GLNXCkbY5Cl53QGji4otCQ.b1ugssErlxrom73eYU+guw;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1euTgKL/sjw.h2Ojb37d+6GAuuL/s6pPsA.3NsywOApt5hDzDKoSV0L4A;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1Wbb6yZ9KFnw.ZJIEdn3mBW4ypwuZfZPGYw.FpOjV90yNMeKFpyg+1RK4g;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1g3E.yS2dqNMVqHCw9iYskwXn6Q.s9N9GQOxzCDySFYav7cm1w;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1zttu1lwzjiv7.ckfjUaXqQXMkGnCwR7kt3w.U7JlMUMSGDHkiBiumoGG3A;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1mq2wFqHFTcBnnL8Gr1y4IN3LZ4BvbMk.+rv1ARg/i0SOvT7Gu1F07w.RP74xs4F7Hh+uEZPFOQhyg;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1jegXu301jC6yk+KKgRWD3OfW.MkEn2F3qFg1BjuWTo6/Evg.tn2Z04tkMNjvNxYjB+kOsA;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:13Pnfm43+KIRum+Q+PytbddNNvMv/yKvRgRFtiMgQ9eQrm9ZUgr4.10VlfB2mlm7Z8Odq68dhcA.JPCiKuZmRG33YyC71nsT3A;}              | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1t75yeA.+DX4+RQ5x2vdoZxGIL6AHA.L+ODzk/kmvvS0y1XjUIfog;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1IwxGxaJrMDk.cZGxLpm/HMHik3VtyfnnRA.r9ei6FMWIiu6r+xV7KU7bQ;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1j/uJEaARH0Bbjw.Ca/o81W1GGOJwSuvW2bQoA.4FtG6yFhry1bQT0Hm1fs4A;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1U2spNM/hGb8.lUCME2B+Eq2AEiEkq3HKvA.xIAyj7DoNbBBPiXG31o3jA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1GH1rgcoiDA.8Tiei8jI/sAOTsi8v0MrXw.XRjFEtxemClx0XAwk+LUCg;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1rM/9oD4q.NW/gVJwh2343SjpWaUOIrA.yxbMhqwY7Plu/hGWrESebw;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1UPU.Cjs1FwYFb1pSAl0GCHgZPA.SIH+S7QYuoifWLQioAUF/g;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1KMJTIm+nOg89.yz0xy6q2VUzGF2l5moLiOQ.ERZO1957xPVaBC99DJ+j3A;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1++GPdqWlcD7mNZXn7OXYDFSUHsY2lfTz.mz2IrNsP5OSqpqgtLH7i8Q.sSimmDOsRf9XXYU3jHdLwA;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1Q1vC7DmneSgash7NCUYTbX161A.dwlKUlOPS7tG4voh68oprw.nf5PGzB5CV4GonN8FZaNfg;}   | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1d9yUgOXl3wlLug/XZ4JYxQSmzzLjRRZ+H7ZOxXLyFso.ukPMfTgypYGih60QG3Auiw.djsOXzB5GUqKEMInItG5AA;}                      | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:155CScQ.QVRHNRtfBKRFnSk7n67wpA.JJvkpVswH6TMhwjjc7YVug;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1qrBj1QMXWZU.Mh3BUbMkI/PvWpR2eYYq4Q.7HjaLeMfRMWCp7giguDRbw;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1mQcJN4zm3TuNsg.C0pPAYNhP7SNdODM3VYQ5w.++NCa+i2XrrnsRGoHAmE5w;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1d+sm75qxZBY./4Ahz+zSuv/eQ4gdl5O+1w.zB32qBW0Sd90ShDNRs4Uxw;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1Fw11CSMiHg.ZtSHgBtv7M/SW6DGCqTAhA.3Auh1x0SRPtESQ6ZF9fObg;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1CndSFeXI.9Uj3LrOb6H/LMPV8KLTcgg.NGBZ7vm9ZbWxhBx7kzuMVw;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1Zlc.qcvHIpQucLeiKizzvL1GiA.iZJIQ8ibmMBaHcM7PofDFQ;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1GJSFHvxt4lwX.0BmNatc180a5To9ui6RA/g.rR+RC2TGfdTakkseuKVs8w;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1C/hkY4Q83FyY4rfd5BDNOIPlxVe3NKHG.ScR+ehb3R3Ar9Q6eYpJr7g.zW7eL0gwPzgBOxVQqdGbfw;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1FMiZSQ/3zOQvmDDhp2AuJTX9PRoz.L5Hgq8OaO47goGTkdCitrg.1nigfFI25T7hmrxhdso0sQ;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1lmIHeVzHxGRL2yg8w7M0fJio9QMlVkrs8iQVQbbLQc2qwQ.wbNPxzm6dpm3Lh5VV/CTag.EvXdL0TqW+eaFTw9r43dUw;}                   | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1BXg3pA.8bkjT7jBWN4DSQ41y6A6Hw.fnns+uotBXtUqIAEkHq1VQ;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1ThUHR2Q0Hi7+.HTKqG5qIsc0s4SMDnoICTA.zV8tHboFwan1jA438giMzQ;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1wniGKlOnJgNVvA.2pC59KnKQQdygsL4sto07Q.6alRF8UckJNoAj/Dtbf47w;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:15kPXYMt1.ISAz67Pzy54hwexCdv3+pQ.jYWAQ3jPgqvyoRPL5HXIUQ;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1IYsmMxu9zTh5.aiTbjpk2W85Dp1+bFiyKsw.k8cTSGEBf/dp0U5FXKb1aw;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1+cvEJRCuyIA.TBOD+nZBchqGg2Qzqrdozg.6+60UlFUvfwJDgYs3xXPkQ;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1ezE.ih/tf+Q1aKlKcSfEIYXRzw.zYnQIcOCLKEEpd8jOhfKlQ;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1wVgL+LAu6p1y.U+fHY5l3sDwZQsAtF2qQLw.EfubdvQvLfmllzJqOEakEw;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1EGDsbUWVdn5OlogE51qaXOugJddIG85A5t6a2g.Erelz9afJexWicVepIh4KA.Lq9DvgPB2pD0dmyoFCVqOQ;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1ZgnAVpEl0DpyzDM9f40.2XwQKP8y/fbuqG2Z3MsPDw.YcFgONQGcBtWtYkUiYykQA;}          | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1u4MrGiVlMxzuGL6j72BgR6JijkrzaLPXaSIfzG7KQPLaSlvBBV3yPzkEQQ.0imy8rXL1Kvu0CAoQVspqQ.RrIKf9GZAYmSUXkCH5sQKA;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1OGk7aA.14w6c62QGEWcEOz4DVlhGQ.EnrlIQim+3k4k0ZoHqsx7g;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:18W2o0krn6XQ.eu3Dih71dpSTpOzRuiZ2lw.uWal5m29r3N7UjopkW5kQA;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:16Wp7Aeo8ku5LFw.AmrK0EdhUVY15/Twv7yF9Q.uEYjebw7XOGKSLWUaNU/OQ;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:100bSSTd3.ZXwU9qQfcikugMbGA77QLg.t5ECSPul89PyP8bt/FUmcw;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1XDC3+Rdh1ZTI.7gPT1QRpBfrqji55y3+Bfg.f7dJ30l0BbEUtgSA5jQBxA;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1ky+/I2ynovM.hZesk/QyRKtXcGvKr8zx8A.lzZqvPoGrfA8x2Aa+/MGdA;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1JC0.W0GpANWEMpTOesN6/UPiBQ.fjmRBs167NdrS2gjgH4+Yg;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:13bENc27owMPv.WCUhYxRjo5KlWli6q5mAGg.T+vrk6QaUkdOtViaOL82Mw;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1rClQL/uKVRgetZXVlWvRfSwXAjM.4EWC5+dH0jpVgyPD//ZcHw.e1ixX/7+CUCTxpXFyg8CNg;}            | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1xIWC2ACHj9njqpMqkuXnJA.4ASOckEtneiIoDWfGEA8jw.HhhRDnxdG4RUjkQykwnavw;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1U6Hs41r6NAs+rdkM9j2gnE6RECKAFJlLzRFHAIJjwqTp.sALQfCH6TzYWu663HgNuLQ.Zwirpz6gQcslPCTF+E/0DQ;}                     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1TnioJQ.mykkRFd+t+JrAkreAz+WkA.xYeowAe/PCpo5OJbC4xkCg;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1DaUCdd4Wi1A.tBvFHxKzys+oE3Idh3n3Hg.euykWpGS42RN2CsCeJ55Bg;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:19ArIMOSNZ5DcAA.tlBOIRFzgbuiT7TRRdYlHA.pFZgiCOx6OohdTwlkbzdXA;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1kD/fBdJu.wBeL+2/plS+LWMKc1sfEyA.PqzYLgl/poyhSBNAuj1lpQ;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:15ULvpzs/Sg.iXBLj2re3NasuYkcC+O3UA.Obr/BHk1/38GLkYJDi7IwA;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1iHgw5BROFGWACeQ./hLH+ssEeL5N9ogEq955lw.Z6yg8KKD1cr2b8cS6KH2LA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1L4U.g7bn6+0WgBxD5HCM32Ek4Q.1e4N7MjnJqi/oJgrZPrc6A;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1wqG3rsQ5dRaO.6HbEcvtbUcy2Q+nJ31ZQtQ.wNX1PAJZRyvyltnCN0U5Xg;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:19MnNizoyBPOAzLg8FVhfF4zAYBXudv8.AvsxKLwZBBLIKi+DDyvsYA.OmSnHlV2hFdKFyLuRGCV/A;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1tetN//QDvZ6kkXEZjAOqYbJTtlAi.1e3ruil4m6Fjn7DDYD+ESw.6UhG/7JdH/Xf6vHUfO3bRA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1fmOkn+k9yU60CErkb/X86gpDiFBMfH3GYJfkgUR3zomQiPSi2OWckrdtk6IW.M9hUWzcwcayPSOUKTxFBHQ.aBf++FBtt2Sxian3IXFFPw;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1UEPGog.dbeCLUT4YHeiBucEGJTvhw.RG7ytkBuzFajGz9N+cq/dA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:13s3qzuY9XqCo.sG+AjHzGJmfXkGpXJFSF4A.m5bjkvuJZCqQ44ViLS1Sbw;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1EiV4vU4FEovnPA.t/oeX0S9lYA0xOJp/BHTBQ.DjbS7fZKgrG3KjxJJ8GbQQ;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1JL109Nju/tY.DQZNAFZe03dJohamNHaAwQ.Z85pE4BvufTtuvG5QjB/gg;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1IzFDzdXTKks.Xd0qmuKdi+6NMIw7kaad3A.Yp0htu8Ki+wwLmf7LBFt9Q;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1+Lr1fptH.n9G7K7ektEVjK7rDULagnQ.tuYARAa+qlqn8DZALhWR+Q;}        | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1l3Y.f5moNlR9G7PKWcPG2lX03g.uX9CLKWRLmx6Px7CG0kbBw;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1zul1YXaPrLUz.OmRCgt40eLm6lQNvHa4QAA.WJauWuPXV6rzdyOvr8WVCg;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1GadprVSh9rSqnWiFPyuQbMZpfLEckw.CsUhMUhWLhycGM88eblCjw.gHtytBj6JBqeOsdP+OSNzQ;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1hcXqduLusClRahvDFHzxj/Jjrvrw.qUoqsCCr2HbQLTePgr6TsQ.LdP1gf+iZqCnk4xrUN3dZw;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:13/+3P99GK5j18sN1IHpMguAqdl5UiqUHmTzbvGNVr0GFmfWZ5iJr+Q.HLIqeXoN9asCaUxAYbesKQ.fPAUKpjUail2ZTSms/HmGA;}           | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1Aa8m0A.K5kuQJUdDcZXvZxeNnu4eA.MMXxeYG0Y3VFI6QrWd4b7Q;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1o/poIMa1zBHS.suZ1x9tjkaGjK8/z8Npbqw.VKLM7l5ObzFDuxUtTts+/Q;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:13f4GUp8BRRPEEA.fkycHDGp7Qvkb6g+JRLIcQ.U/eSF2zbaK4Gvu3xtry4uQ;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1LQS42V058+w.DTFLdm/wDozzpDuFZdWw3w.GVOdf0EFlAHHlYxn0+tiZA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1fGlUSOpm/XwvtQ.6q6ZBzjgs3UN1DkkgJ0yDg.ATANlP34CcrWoXcGkx955A;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1nCV4nbPL4lE.6QNhHrkNiAfUPH0PaMkgoQ.Dhi1JYtD9QiHskfZ8pPzRw;}     | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:1Z6g.2qWLlCZMfQ1FsFCtcFcfUA.MC2yG38qCwkJdGSjbbA+/w;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:109f8sdmJGgEK.g2BSxIfCR06R1i7GMiLunA.slLWlhtTznNTukXWGAcBZA;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1Z+ia/cTFT3WfRU38nvv5n96bl5s.X3UQjQiDflFhxxQq25hdtA.P4nWkFXOgpx02D2PH48r5Q;}            | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1ER3/yoDgvpwgG9ZCgug.No7DPNL4tbvuf9b8kLaP4w.Mjr4eEpQEXZEG7HPjUuI1g;}          | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1VHA+VIFhVaHJnpwPL+mycM69CIMGej6q8q7WoM7KgQJ4Pf97e/TCYEKllA21Fr7v.RFlhut8jx/U+e6u6ECm3Xw.VzUlr0tADKlBghY5vBQWVQ;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1bKf2mQ.XCevE2tOPicHvaWja1LeUg./qZ/brRXdSTNqS3WlRcoHQ;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:14oxUfa6+lsE.Dv6B4vXdgbsC2N87sECjYQ.Yx7Cw9+oxv2Fez5s7yc/Pg;}
 ##P2{#K:demompl.mykeyforapp1;;,#I:"/orig_cif_number";,#D:1sEi/R6iZ197nSA.8gFXU3PTxD6f8ShUJs2O8A.QtjNJ1q0dR34hEcq5F52mw;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/gender";,#D:1ju8U9U3j.2jpSE3Vl7q0ptLTvk0sXAQ.GsqNxhYER7nWrb+SKM+6rQ;}    | ##P2{#K:demompl.mykeyforapp1;;,#I:"/first_name";,#D:1pw9bDXktLQ.DVbJIoxnU9aHDQwGXV75Jw.7E0liAqpI1uLcywfP0aXVQ;}         | ##P2{#K:demompl.mykeyforapp1;;,#I:"/last_name";,#D:1J/qgV7A18w.n9zhu0VKr3bu6kBvY34SCA.NAYJdnFE/ikbTxd3bc1x9Q;}      | ##P2{#K:demompl.mykeyforapp1;;,#I:"/age";,#D:14K8.NqJLfrfqmCczV3UUcLylBw.zbJ1lJaxLmLFj86g1YQQxg;Int} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/sin";,#D:1V+XeDqx1OdaO.eZiV9zbemmMFYH5zUKE+qg.NVXJzP3ThdiDaWeesvXpIw;Long} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/email";,#D:1R579aHV4v+v+mwQa/LJM6fwqXQoAHlTg.PKHhwCuOuuDSsMLR9RqhUg.PU+H1FR/hABa06RCR+NJpw;}       | ##P2{#K:demompl.mykeyforapp1;;,#I:"/phone";,#D:1k2sJzzsG7ZMQpmwWnEguRNxZ40KZ.hTRnlpMZngNVFYhRAudNbg.a7AN1f/ytWkH7tpT6Fi9Xw;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/mailing_address";,#D:1e5rnJakmufQurpsdIt/dY9auf2FZWN+znTnVlkOfhmdVCXesiiPe.8JdYaByxQW87T3WrRlBBaw.NTZ+A6a+D8OlkIXqr9BMNw;}             | ##P2{#K:demompl.mykeyforapp1;;,#I:"/prov_abbr";,#D:1soSXIA.onaYcf7KgLYQXzwhH8fx1g.Lta0UpSL/IKw/Y5Bso9mfA;} | ##P2{#K:demompl.mykeyforapp1;;,#I:"/postal_code";,#D:1DzYsCJYw2rM.LmOujmn9xDAg1Un++6jm4Q.0nV5nhNiZX0LQhRgenkd3g;}
10 rows selected (0.202 seconds)
Beeline version 1.2.1 by Apache Hive
Closing: 0: jdbc:postgresql://ec2-35-180-97-38.eu-west-3.compute.amazonaws.com/userdb
```
:exclamation: Off-course we can't understand anything, we just query a TDO directly. What we see is security mecanism and the data together. As a TDO, data is secured no matter where it is, and no matter the security of environment the data is crossing.

There is only one way to get access to the data of the TDO in the clear, this is via IBM Data Privacy Passports.

## 2. Protection and then enforcement of the data with IBM Data Privacy Passports
:white_check_mark: **Objective:** Data is protected at the exfiltration point and off the platform, and data is enforced at the consumption point.

You can find below how a JDBC application experiences an SQL Query to a TDO via IBM Data Privacy Passports.
* **(1)** User connect to an URL pointing to IBM Data Privacy Passports. For such connection, it is mandatory to provide valid credentials, the name of the TDO, driver name, ... 
* **(2)** IBM Data Privacy Passports checks in the policy the DbViews section to find the appropriate JDBC connection profile to SQL query the TDO.
* **(3a)** DBMS sends back SQL query output to IBM Data Privacy Passports as it is (protected data).
* **(3b)** The Passport Controller directly enforces the data (according to the policy) coming from the source DBMS.
* **(3c)** IBM Data Privacy Passports via the Passport Controller send back the data (enforced data) to the user.

![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protect-then-enforce.png?raw=true)

Let's SQL query a TDO via IBM Data Privacy Passports with different users.

### 2.1 SQL query as a Data Owner (DO) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.20:10010" -n DOUser -p DOUser -e "select * from awspostgresql.protected_customer LIMIT 10;"
```
Expected output is:
```
Connecting to jdbc:hive2://10.3.58.20:10010
Connected to: Spark SQL (version 2.4.4)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+------------------+---------+-------------+-------------+------+------------+---------------------------+--------------------+------------------------------------------------------+------------+--------------+--+
| orig_cif_number  | gender  | first_name  |  last_name  | age  |    sin     |           email           |       phone        |                   mailing_address                    | prov_abbr  | postal_code  |
+------------------+---------+-------------+-------------+------+------------+---------------------------+--------------------+------------------------------------------------------+------------+--------------+--+
| 1000015367       | Male    | Alexander   | Edwards     | 20   | 295191119  | xhansen@yahoo.com         | 758 331 7845       | 2230 Steele Knoll Suite 136, Andersonview            | NB         | P4M6K4       |
| 1000002123       | Male    | Gregory     | Hayes       | 59   | 123411260  | yhughes@rivera-lewis.biz  | 1 (983) 186-1803   | 39540 Alvarez Lock Suite 992, West Rhondashire       | MB         | C1X2G8       |
| 1000009222       | Male    | James       | Kelley      | 61   | 996764772  | critter@bryant.com        | 1 (686) 971-6309   | 43354 Jennifer Course, Jeremiahside                  | NB         | B6K 6S7      |
| 1000004121       | Male    | Gregory     | Murphy      | 34   | 791289389  | qtaylor@hotmail.com       | 357 263 9437       | 790 Murray Center, Jamesville                        | NL         | R4K 6G4      |
| 1000011197       | Male    | Peter       | Thompson    | 37   | 306348876  | wigginsamanda@yahoo.com   | (619) 871-8863     | 60273 Jennifer Estates, South Normamouth             | YT         | R7C 2T7      |
| 1000012931       | Male    | Justin      | Krause      | 44   | 215745453  | jamescarey@yahoo.com      | +1 (469) 771-5484  | 4390 Julie Neck, New Alexander                       | NS         | G1Y 9J3      |
| 1000019925       | Male    | Ronald      | Villarreal  | 28   | 241592879  | wongjoseph@knapp.com      | 1-812-658-2264     | 7427 Ryan Groves, North Justinhaven                  | NT         | R6M 5Y2      |
| 1000007911       | Male    | Michael     | Nelson      | 57   | 871796960  | sstevens@yahoo.com        | 1 (182) 338-1247   | 9226 Alvarado Isle, Port Jake                        | NV         | E6C 6S1      |
| 1000008368       | Female  | Brooke      | Clark       | 48   | 690448117  | smithnichole@gmail.com    | 615-546-6937       | 47171 Kristopher Parkways Suite 711, Jefferyborough  | NB         | N7B5C7       |
| 1000012225       | Female  | Hailey      | Smith       | 46   | 683477466  | mrobinson@brown.com       | 341-664-7817       | 14770 Tonya Hollow Apt. 205, Jacksonburgh            | YT         | V8E2E5       |
+------------------+---------+-------------+-------------+------+------------+---------------------------+--------------------+------------------------------------------------------+------------+--------------+--+
10 rows selected (15.402 seconds)
Beeline version 1.2.1 by Apache Hive
Closing: 0: jdbc:hive2://10.3.58.20:10010
```

### 2.2 SQL query as a Data Administrator (DA) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.20:10010" -n DAUser -p DAUser -e "select * from awspostgresql.protected_customer LIMIT 10;"
```
Expected output is:
```
Connecting to jdbc:hive2://10.3.58.20:10010
Connected to: Spark SQL (version 2.4.4)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+------------------+---------+-------------+------------+--------+--------+--------+--------+------------------+------------+--------------+--+
| orig_cif_number  | gender  | first_name  | last_name  |  age   |  sin   | email  | phone  | mailing_address  | prov_abbr  | postal_code  |
+------------------+---------+-------------+------------+--------+--------+--------+--------+------------------+------------+--------------+--+
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
| XXXXXXXXXX       | XXXXX   | XXXXX       | XXXXX      | XXXXX  | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | XXXXX      | ZZZZZ        |
+------------------+---------+-------------+------------+--------+--------+--------+--------+------------------+------------+--------------+--+
10 rows selected (15.856 seconds)
Beeline version 1.2.1 by Apache Hive
Closing: 0: jdbc:hive2://10.3.58.20:10010
```

### 2.3 SQL query as a Data Consummer (App1) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.20:10010" -n App1User -p App1User -e "select * from awspostgresql.protected_customer LIMIT 10;"
```
Expected output is:
```
Connecting to jdbc:hive2://10.3.58.20:10010
Connected to: Spark SQL (version 2.4.4)
Driver: Hive JDBC (version 1.2.1)
Transaction isolation: TRANSACTION_REPEATABLE_READ
+------------------+---------+-------------+------------+------+--------+--------+--------+------------------+------------+--------------+--+
| orig_cif_number  | gender  | first_name  | last_name  | age  |  sin   | email  | phone  | mailing_address  | prov_abbr  | postal_code  |
+------------------+---------+-------------+------------+------+--------+--------+--------+------------------+------------+--------------+--+
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 20   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NB         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 59   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | MB         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 61   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NB         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 34   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NL         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 37   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | YT         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 44   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NS         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 28   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NT         | ZZZZZ        |
| XXXXXXXXXX       | Male    | XXXXX       | XXXXX      | 57   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NV         | ZZZZZ        |
| XXXXXXXXXX       | Female  | XXXXX       | XXXXX      | 48   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | NB         | ZZZZZ        |
| XXXXXXXXXX       | Female  | XXXXX       | XXXXX      | 46   | 99999  | XXXXX  | 99999  | XXXXX@XXXXX      | YT         | ZZZZZ        |
+------------------+---------+-------------+------------+------+--------+--------+--------+------------------+------------+--------------+--+
10 rows selected (15.702 seconds)
Beeline version 1.2.1 by Apache Hive
Closing: 0: jdbc:hive2://10.3.58.20:10010
```

### 2.4 SQL query as an unknow user (Test) of a TDO via IBM Data Privacy Passports:

:computer: Let's experience what data looks like in a TDO:

```
beeline -u "jdbc:hive2://10.3.58.20:10010" -n Test -p Test -e "select * from awspostgresql.protected_customer LIMIT 10;"
```
Expected output is:
```
Connecting to jdbc:hive2://10.3.58.20:10010
Error: Could not open client transport with JDBC Uri: jdbc:hive2://10.3.58.20:10010: Peer indicated failure: Error validating the login (state=08S01,code=0)
No current connection
Error: Could not open client transport with JDBC Uri: jdbc:hive2://10.3.58.20:10010: Peer indicated failure: Error validating the login (state=08S01,code=0)
```

# 3. Conclusions and next steps

You can find below a summary of the different Protection use cases.
![alt-text](https://github.com/guikarai/IBMDPP/blob/master/Protection-matrix.png?raw=true)

This close the Data Protection chapter. Next chapter in this hands-on labs is [Audit trail](https://github.com/guikarai/IBMDPP/blob/master/audit-trail.md).

