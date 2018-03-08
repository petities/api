# api
The application interface for a website built on the petitions.eu code

Other systems can connect with the database of petitions through this interface. For example those used in the councils of municipalities, but also campaigners who want signatories to sign through a campaign website. 

Please e-mail webmaster@petities.nl if you find bugs or have questions about the api not answered here. 

## Structure

There are three important tables:

1. petitions
2. signatures 
3. organisations.

Petitions and signatures can be queried. Petitions can only be added through the webinterface, but certain properties to existing petitions (like status) can be changed by certain users. Signatures can be added, but not confirmed or modified through this interface. Signatures are confirmed by following a unique link from an e-mail send to the e-mail address entered by the signatory. There the data can be modified by the signatory.

Organisations receive and answer to petitions.
