---
Author: Gu Jianjie
Data: 2017-08-23T00:00:00.000Z
---

# UAES Secured Flashing Improvement \(MG1U Platform\)

**Motivation**

* Further sercured flashing improvement 
* Minize technical gap between RB solution

**Key Points**

* Public Key -&gt; CVC
* 1-Level Signature Verification -&gt; 2-Level Signature Verification

**Detailed Description**

* Old solution

  * Only 1 Public Key is stored in the BootCtrl area

  * Signatures of ASW or DS block are stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.

---

![](/assets/old_static.png)

* New solution
  * Only 1 Public Key is stored in the BootCtrl area.
  * Signatures of ASW or DS block are stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.

![](/assets/new_static.png)

