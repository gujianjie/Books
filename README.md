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

#### _Old Solution_

* **Static design of signature in old solution**

![](/assets/old_static.png)

  * Only 1 Public Key is stored in the BootCtrl area.
  * Signatures of ASW or DS block are stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.


* **Dynamic design of signature in old solution**

  * The download process of a logic block (ASW or DS) normally consists of 5 steps. 
    * Request to enter 'Programming Session'
    * Reqeust to get Security Access authentication
    * Request to erase corresponding logic block (ASW or DS)
    * Transfer new data into the controller
    * Request TransferExit, meanwhile, chenk the integration of the downloaded content

  ![](/assets/basic_download_process.png)

  * Detailed in TransferData process
      * TransferData service checks if ValidPattern (see InfoBlock struct) is included in the transferred package. If ValidPattern is detected, it's to be stored temperally into the ram instead of flash
      * The rest of the data is transferred into the flash buffer, then written into flash via Fls LLD in the background task    
 
  ![](/assets/dynamic_transferdata_process.png)

  * Detailed in TransferExit process
    *
---

#### _New Solution_

* **Static design in new solution**

<div>
  ![](/assets/new_static.png)
</div>

* Dynamic design of signature verification in new solution



