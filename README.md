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
  * Signature of ASW/DS block is stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.


* **Dynamic design of signature in old solution**

  * The download process of a logic block (ASW or DS) normally consists of 5 steps. 
    * Request to enter 'Programming Session'
    * Reqeust to get Security Access authentication
    * Request to erase corresponding logic block (ASW or DS)
    * Transfer new data into the controller
    * Request TransferExit, meanwhile, chenk the integration of the downloaded content

  ![](/assets/basic_download_process.png)

  * Detailed in TransferData process
      * TransferData service checks if ValidPattern (see InfoBlock struct) is included in the transferred package. If ValidPattern is detected, it's to be stored temporarily into the ram instead of flash
      * The rest of the data is transferred into the flash buffer, then written into flash via Fls LLD in the background task    
 
  ![](/assets/dynamic_transferdata_process.png)

  * Detailed in TransferExit process
    * BootCtrl/CB gets root public key stored in the Bootctrl via InfoTab of BootCtrl
    * BootCtrl/CB verfies Signature stored in the Epilog, which has been programmed into Flash in TranferData process, with the root public key
    * If the Signature is valid, the ValidPattern, which has been temporarily stored in ram in the TransferData process, will be programmed into Flash. Otherwise, the flashing process is aborted. The corresponding block will be marked as invalid in the next PowerOn Test.
    
  ![](/assets/dynamic_transferExit_process.png)
---

#### _New Solution_

* **Static design in the new solution**

  ![](/assets/new_static.png)
  
  * A RootCVC certification is stored in the Epilog of BootCtrl. The RootCVC certification is a self-signd certification. The RootPublicKey is included in the RootCVC.
  * A ProjectCVC(with ProjectPublicKey) is stroed in the Epiloog of CB/ASW/DS. The ProjectCVC is signed by the backend KMS with the RootPrivateKey. RootPrivateKey is the paired private key of RootPublicKey
  * Signature of CB/ASW/DS block is stored in corresponding Epilog area. The signature is generated with the ProjectPrivateKey stored in the backend KMS server or offline eToken.
  

* **Dynamic design of signature verification in the new solution**

  * No change in the 5-step download process
  * No change in the TransferData process
  * Detailed in TransferExit process in the new solution
    * BootCtrl/CB gets RootCVC certification stored in the Bootctrl's Epilog according to the defined Epilog structure.
    * BootCtrl/CB extracts RootPublicKey from RootCVC according the CVC encoding format(TLV)
    * BootCtrl/CB verifies the ProjectCVC stored in the Epliog of CB/ASW/DS with the RootPublicKey  (CB can only verifies ProjectCVC of ASW/DS). Additionally, CB can also checks the project related tags (e.g. Project Id or SubjectName) in the ProjectCVC of ASW/DS with its own CBProjectCVC for compatiblity purpose. (See detaild CVC design)
    * If ProjectCVC is valid, BootCtrl/CB can extract the ProjectPublicKey for ProjectCVC, and trust it for further usage. Otherwise, the flashing process is aborted.
    * BootCtrl/CB verifies the Signature stored in the Epilog, which has been programmed into Flash in TranferData process, with the ProjectPublicKey.
    * If the Signature is valid, the ValidPattern, which has been temporarily stored in ram in the TransferData process, will be programmed into Flash. Otherwise, the flashing process is aborted. The corresponding block will be marked as invalid in the next PowerOn Test.

  ![](/assets/dynamic_transferExit_process_new.png)
