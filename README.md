---
Author: Gu Jianjie
Data: 2017-08-23T00:00:00.000Z
---

# UAES Secured Flashing Improvement \(MG1U Platform\)

**Motivation**

* Further sercured flashing improvement 
* Minize technical gap between RB solution

---

**Key Points**

* Raw Public Key -&gt; CVC Certification
* 1-Level Signature Verification -&gt; 2-Level Signature Verification

---

**Detailed Description**

* Old solution

  * Only 1 Public Key is stored in the BootCtrl area
  * Signatures of ASW or DS block are stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.
  

* Static design of signature in old solution
  ![](/assets/old_static.png)

* Dynamic design of signature in old solution

---

* New solution
  * Only 1 Public Key is stored in the BootCtrl area.
  * Signatures of ASW or DS block are stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.


* Static design in new solution
  ![](/assets/new_static.png)

* Dynamic design of signature verification in new solution



