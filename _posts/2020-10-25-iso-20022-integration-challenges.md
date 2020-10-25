---
layout: post
disqus: true
fbcomments: false
published: true
title: ISO 20022 integration challenges
---
ISO 20022 is a global standard used to facilitate electronic data interchange between financial institutions. 
## Benefits
It provides many benefits not limited to  
* Flexibility - adapt to changes, technology changes and innovation
* Harmonisation - across payment systems round the world
* Compliance and regulation - structured data makes it easier to detect fraud and prevent financial crime
* Efficient routing and increased resilience
* Increased straight-through processing rates
* Richer data - for improved decision making

![ISO20022 Integration Challenges]({{site.baseurl}}/media/iso20022challenges.png)

## Benefits come at a cost

### Complex Schemas

* The need to be a global standard means message schemas are big, complex and deeply nested. The ISO 20022 message are designed to handle all kinds of payment scenarios, no matter how esoteric. 
* Lets look at a typical message. The _FinancialInstitutionCreditTransfer_ (pacs.009.001.06) message is sent by a debtor financial institution to a creditor financial institution, directly or through other agents and/or a payment clearing and settlement system.
* [FinancialInstitutionCreditTransfer (pacs.009.001.06)](http://paytoolz.com/pacs.009.001.06.doc.svg) Hover on elements for documentation 
* Names like _FICdtTrf_ are not exactly memorable. ISO has tried to find a middle ground between cryptic codes such as F54, and full 
names such as _FIToFICustomerCreditTransfer_, but they are hard to remember and get exactly right each time. 
* One has to use full path such as _Document.FIToFICstmrdtTrf.GrpHdr.Sttlmlnf.lnstgRmbrsmntAgt.Finlnstnld.CIrSystbld.CIrSysId.Cd_ to access any element. There are no shortcuts.

 ![Deeply nested structures]({{site.baseurl}}/media/nestedstructure.png)

### Multiple message types and variants
* There are many message types, their local variants and versions each with its own schema.  At the last count there are 693 on the ISO 20022 website in the payments domain alone, not including previous versions.
* If a process that needs to access the Sending institutions SWIFT code (BICFI). A business analyst has to go through multiple message definition reports and specify something similar to 
* This is time consuming and is repeated many times. The application logic also needs to implement the specific logic.

![Multiple message schemas]({{site.baseurl}}/media/vocabulary.png)

### XML and XML Tooling
 * XML is primarily designed for information interchange (messages) and is indispensible for implementing a global standard. Since XML is primarily a syntax for messages, it is hard to query and manipulate efficiently. 
 * The world has moved on from XML to other lightweight weight formats such as JSON. As a result, any innovation and tooling support improvements for XML have come to a standstill. Many XML open source projects are stuck in a limbo. 
  
  ![xmlvsjson]({{site.baseurl}}/media/xmlvsjson.png)
 * Most existing methods to manipulate XML are not type safe and you need to ensure the XML you create is compliant with the ISO 20022 schema. 

### ISO schema generation standard [ISO 20022-4:2013](https://www.iso.org/standard/55008.html)
 * The schema generation standard adopted by ISO has some quirks that makes ISO 20022 message manipulation even harder
 * Schemas are not documented
 * Each message has a separate namespace making translations and conversions between versions harder
 * Complex types in schemas utilise <xs:sequence> tags which require items in a specific order instead of <xs:all>
 * Currency types use XML attributes for currency code making access and manipulation unnecessarily difficult.
 
## PayToolz
We at [PayToolz](http://www.paytoolz.com) have come up with a set of tools to help with ISO 20022 integration and address each of the issues listed above. These tools are directly generated using the ISO 20022 metadata repository which ensures compliance with the standard is out of the box. More on our tools in part 2. 
 
![PayToolz Toolkit]({{site.baseurl}}/media/toolgeneration.png)
