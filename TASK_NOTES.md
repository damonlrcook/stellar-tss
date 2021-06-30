# Task.io installation notes and observations
We’re using this file to note down tweaks to installation processes as well as conventions we plan to use as we roll this out.

This can be combined with the main repo files if the community finds it useful.

## Notes on installation process
Some aspects of the installation process are not the easiest to understand for a first attempt with fresh accounts in AWS, Cloudflare and Serverless. This section aims to clear up certain parts of the installation process to enable an easier understanding of the reference implementation.

### wrangler.toml
This file requires multiple changes to certain values within the file to enable the turret to be set up. The reference implentation documents the changes to this document and there are no extra steps that are documented in the intial setup of the wrangler directory.

### stellar.toml
This document is modified directly after the implementation of the wrangler.toml file, to clarify the only change that is made to this document is the replacing of the turret address which has the comment after it of self. This is to be replaced with the address of the turret that you are setting up. The code can be viewed below.

    [TSS]
    TURRETS = [
      'GB4OYM7TQTJSROWXHOJLKAX2IJ2QN4I6S6GCJH4MGWVTAO5Q5DPNADXX', # self
      'GAMGW7ZESFF2KVGL7EMBBIAXTFAOF3MNI2JBWSPLTGMAKOSVLHDNZMA2', # EverLife.AI
      'GAATC6A7OOY7P36FTDFEHKHG7OS72A3NTSGM4RA6HJ5GKHUOEVRWGM76', # Script3
      'GARBNUK4IZDGKDJTIEWAL6FTY3UDRSRV275X4H5JS2PGCKTZZ3NDYU5U', # Litemint
      'GDZACR7XAJWU3JLJ5TVDOLCVB5DCIY2K7PHDR7ODGCEQTNVENOPHLCF7', # Atemosta
      'GA7FCNDCYS2N4XQER6P2KHK4FDVVM2PJCM3PODAZXC35QNJR5WINT6GO', # Zig3
      'GATCMFLSJYFQ3XVNMHXKWHVUYKGQ4FDRD6U7VXSDEIUDI7MVTK2SON5Q', # Ralphilius
    ]

### Serverless Setup
It is mentioned during the initial implementation that a Serverless account is setup for the reference implentation to be used. What is not mentioned in the establishing of a Serverless account is that it needs to be paired with your AWS console account as well. This is done by adding AWS as a provider within the profile settings section of the Severless account. Once it has been connected to the account it is necessary that the following step is to create an application within Serverless. 

Once the create app button has been clicked the Serverless framework needs to be selected. Following this you will be presented with a screen that will request you to input an app name and a service name. The recomended naming conventions for this are lsited below. 

### serverless.yml
The reference implementation fails to mention the need to adjust values within the document, these values relate back to the ones that were created in Serverless.

    org: YOUR-ORGANISATION
    app: tss-NETWORK-IDENTIFIER
    service: tss-serverless

# Naming Conventions
## Turrets
tss-**NETWORK**-**IDENTIFIER**.taskio.workers.dev where

* Network = TESTNET or PUBLIC
* Identifier = Unique name of Turret server, thic can be a number or other naming convention that is useful.
## Serverless Apps
**Organisation:** Your Organisation Username (predetermined by the unsername of the Serverless Account)

**App:** tss-**NETWORK**-**IDENTIFIER**

**Service:** tss-serverless

* Network = TESTNET or PUBLIC
* Identifier = Unique name of Turret server, thic can be a number or other naming convention that is useful.  
  
