---
Author: Gu Jianjie
Data: 2017-08-23T00:00:00.000Z
---

# UAES Secured Flashing Improvement \(MG1U Platform\)

**Motivation**

* Further sercured flashing improvement 
* Minize technical gap between UAES solution and RB solution

---

**Key Points**

* Raw Public Key -&gt; CV Certificates
* 1-Level Signature Verification -&gt; 2-Level Signature Verification

---

**Detailed Description**

#### _Old Solution_

* **Static design of signature in old solution**

  ![](/assets/old_static.png)

  * Only 1 Public Key is stored in the BootCtrl area.
  * Signature of ASW/DS block is stored in corresponding Epilog area. The signature is generated with the corresponding PrivateKey stored in the backend KMS server or offline eToken.

* **Dynamic design of signature in old solution**

  * The download process of a logic block \(ASW or DS\) normally consists of 5 steps. 
    * Request to enter 'Programming Session'
    * Reqeust to get Security Access authentication
    * Request to erase corresponding logic block \(ASW or DS\)
    * Transfer new data into the controller
    * Request TransferExit, meanwhile, chenk the integration of the downloaded content

  ![](/assets/basic_download_process.png)

  * Detailed in TransferData process
    * TransferData service checks if ValidPattern \(see InfoBlock struct\) is included in the transferred package. If ValidPattern is detected, it's to be stored temporarily into the ram instead of flash
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
  * A ProjectCVC\(with ProjectPublicKey\) is stroed in the Epiloog of CB/ASW/DS. The ProjectCVC is signed by the backend KMS with the RootPrivateKey. RootPrivateKey is the paired private key of RootPublicKey
  * Signature of CB/ASW/DS block is stored in corresponding Epilog area. The signature is generated with the ProjectPrivateKey stored in the backend KMS server or offline eToken.

* **Dynamic design of signature verification in the new solution**

  * No change in the 5-step download process
  * No change in the TransferData process
  * Detailed in TransferExit process in the new solution
    * BootCtrl/CB gets RootCVC certification stored in the Bootctrl's Epilog according to the defined Epilog structure.
    * BootCtrl/CB extracts RootPublicKey from RootCVC according the CVC data structure\(ASN.1\)
    * BootCtrl/CB verifies the ProjectCVC stored in the Epliog of CB/ASW/DS with the RootPublicKey  \(CB can only verifies ProjectCVC of ASW/DS\). Additionally, CB can also checks the project related tags \(e.g. Project Id or SubjectName\) in the ProjectCVC of ASW/DS with its own CBProjectCVC for compatiblity purpose. \(See detaild CVC design\)
    * If ProjectCVC is valid, BootCtrl/CB can extract the ProjectPublicKey for ProjectCVC, and trust it for further usage. Otherwise, the flashing process is aborted.
    * BootCtrl/CB verifies the Signature stored in the Epilog, which has been programmed into Flash in TranferData process, with the ProjectPublicKey.
    * If the Signature is valid, the ValidPattern, which has been temporarily stored in ram in the TransferData process, will be programmed into Flash. Otherwise, the flashing process is aborted. The corresponding block will be marked as invalid in the next PowerOn Test.

  ![](/assets/dynamic_transferExit_process_new.png)

---

#### _Detailed CVC design_

The storage in the embedded ECUs is normally limitied, so X.509 certificates are not used, CV certificates according to ISO-7816 is used instead. The certificate profile is specified in the following table.

![](/assets/CVC_Profile_overview.png)

The encoding of data structures defined according to ASN.1 is described in X.690. The tags,lengths,and values used in the CVC is defined in the following table.

![](/assets/Overview_dataObject_CVC.png)

The data objects contained in an **RSA public key** are shown in the following table. The order of the data objects is fxed.

![](/assets/PublicKey_Profile.png)

The **Certifcation Authority Reference** is used to identify the public key to be used to verify the signature of the certifcation authority. The Certifcation Authority Reference MUST be equal to the Certifcate Holder Reference in the corresponding certifcate of the certifcation authority.

The **Certifcate Holder Reference** is used to identify the public key contained in the certifcate.

The role and authorization of the certifcate holder SHALL be encoded in the **Certifcate Holder Authorization Template**. This template is a sequence that consists of the following data objects:  
1. An object identifer that specifes the terminal type and the format of the template.  
2. A discretionary data object that encodes the relative authorization, i.e. the role and authorization of the certifcate holder relative to the certifcation authority.

* **Example of RootCVC and UserCVC**

![](/assets/RootCVC_UserCVC_Example.png)

---

### _Definitions in the UAES MG1U platform solution_

* **Certificate Profile Identifier** is fixed as 0x00.
* **Public Key** should be RSA2048 format. The **Public Exponent** is fixed as 0x10001.
* The **Certificate Effective Date** should be the date when the certificate is generated. The **Certificate Expiration Date** could be defined, but might not be checked in the ECUs.
* The **Certifcate Extensions** should not be used.
* The **Certificate Authority Reference** and The **Certificate Holder Reference** use the data structure in the following table. The data lenth is limited at 16 bytes. The data element could be further detailized and extended. Ojbect Id is currently not defined and checked. The RootCVC is a self-signed certificate, so the Certificate Authority Reference and Certificate Holder Reference of RootCVC has the same value.

![](/assets/CertificateReference.png)

* The **Discretionary Data** in the **Certificate Holder Authorization Template** should only occupy 1 byte. The definiton of role and access right difinition should be compatible with the following table:

![](/assets/DiscretionaryData_Role_AccessRight.png)

* **Certificate Validation Process**

UserCVC should be verified according to the following process: 

* The UserCVC could be read out from the Epilog of the programmed logical block (CB/ASW/DS)
* The CVC  is stored with ASN.1 encoding. The user should perform format-check and phrase the CVC content
* The Certificate Profile Identifier should be check. The value MUST be 0.
* The Certificate Authority Reference of the UserCVC should be compared with the Certificate Holder Reference of the RootCVC stored in BootCtrl
* The rational of Role and The Access Right of the UserCVC should be checked.
* The Signature of UserCVC should be verified with the Root PublicKey stored in the RootCVC in the BootCtrl.
* 'Memlay Signature Check' bit of the Access Right should be set in the 'Secured Programming' scenario. The 'TSW Signature Check' bit shoud be set in the TSW downloading scenario.
* User PublicKey can only be trusted after passing all the checks, and can only be used in the targeted scenario defined in the Access Right

![](/assets/CVCValidationProcess.png)

* **SwitchOver**
The SwitchOver certificate is us
---

### _eToken Application Process_

* **eToken ApplicationForm**

![](/assets/eTokenApplicationForm_cn.png)

![](/assets/eTokenApplictionProcess.png)

### _Software Release Process_

TBD



