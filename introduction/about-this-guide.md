---
title: "About this Guide"
excerpt: ""
---
This guide is ICON's primary developer documentation and answers the questions that most developers have when learning to develop on ICON. You will find step-by-step instructions, examples, guidelines, and a comprehensive reference. This guide is continually updated with new topics, edits to existing material, and improvements in how the material is presented. After learning the basics on ICON, you can use this guide as a springboard into more advanced topics.

How to use this guide : 

- Are you curious why ICON matters? Start with [Introduction](#introduction).
- Do you want to develop a Smart Contract? Go to [SCORE](#score).
- DApp developers, you will definitely need to interact with your Smart Contract and send transactions to the ICON network from your DApp. Read [ICON SDKs](#icon-sdks). ICON wallet (ICONex) also exposes a couple of useful APIs to sign your transaction on your behalf. [ICONex Connect](#iconex-connect) is for that.  
- Interested in being a P-Rep? Everything about operating a P-Rep node is here, [Running an ICON Node](#running-an-icon-node). 
- Curious about ICONLOOP's enterprise solution? [Enterprise Solution - loopchain](#enterprise-solution---loopchain)
- [References](#references) - Want to go deeper? A comprehensive collection of how-to guides, protocol specifications, design specifications, and user manuals. 

## Documentation Structure

The ICON technical documentation is organized as follows.  

### Introduction 

If you are new to ICON, this is a good starting point. You will learn the motivation and the design goals of the ICON project. We will convince you why ICON is technically different from other blockchain networks.   

- **What is ICON**
- **ICON Key Concepts**: Account, transaction, transaction fees, different types of ICON nodes, parallel processing, P-Rep election, ICON incentive scoring system, and the interchain protocol are explained here. 
- **The ICON Network**: Will show how the ICON network is logically structured across the world. 

### ICON Application Services

No platform is complete without supporting services. 

- **ICONest**: Token creation and Crowdsale platform.   
- **ICONex**: ICONex is the official ICON wallet application. Chrome, Android, and iOS applications are available. 
- **Tracker**: ICON block and transaction explorer. 
- **ICON ID**: Decentralized ID on the blockchain.
- **User Manuals**: Reference manuals for detailed functions of the above applications. 

### SCORE

This is where the fun begins. Smart Contracts are the core building blocks of every blockchain application. In this section, you will get everything you need to know to write, test and deploy a Smart Contract (SCORE) on ICON.

- **SCORE Quickstart**: No previous ICON knowledge or experience required. As long as you are a developer and proficient with Python, Docker, and Linux, following this guideline will lead you to deploy and run your first SCORE on the ICON testnet.
- **SCORE Development Guide**:  All the SCORE development tools and syntax are explained here. 
- **SCORE Audit**: Please make sure you comprehend the audit checklist. This is the SCORE coding patterns that the platform mandates. 
- **Sample SCOREs**: Reference implementation and explanation.

### T-Bears

T-Bears is a SCORE development suite. In order to write a Smart Contract on ICON, you need to install T-Bears first. T-Bears installs the `iconservice` python package which is required to write a SCORE, CLI tools to create, run, test, and deploy a SCORE, and the emulated node environment that runs on your local machine.

### ICON SDKs

ICON SDK is a library that implements the JSON-RPC v3 protocol to interact with the ICON nodes and execute the SCORE functions. ICON officially supports Java, Python, JavaScript, and Swift.

### ICONex Connect

ICONex Connect is a protocol that signs and sends a transaction on behalf of an application. The ICONex app will prompt the user to enter a password.

### Running an ICON node

- **P-Rep**: If you are a P-Rep or a P-Rep candidate, please read this section. This section contains all the guidelines from node installation to troubleshooting. 
- **Citizen Node**: Sometimes you need to run your own citizen node for various reasons, e.g. security or performance. This section explains the installation and configuration of the citizen node.

### Enterprise Solution - loopchain

We will briefly introduce loopchain, ICONLOOP's enterprise blockchain solution. Inter-connecting ICON mainnet with the private loopchain networks is one of ICON's big milestones. The loopchain engine contains all the technical components of ICON with the addition of enterprise-specific features. 

### References

- **How-to** : How-to guidelines to achieve frequently-questioned tasks. 
- **Reference Manuals** : References and technical specifications. 
- **User Manuals** : ICON applications' user manuals

<br>

Besides technical documentation, the ICON Developers Portal includes the following resources:
## Announcement

Developer-related news such as workshops, meetups, ICON's key milestones, and new specification proposals will be announced here.    

## Forum

A code-level discussion forum. We hope the ICON developer community will grow. You don't need to sign up to post a question or share an idea, and your email is not visible to the public. Please be respectful to the community. Any irrelevant or offensive posts can be removed by the forum moderators without notice. 

## Support

Various channels where you can send your feedback, report a bug, and speak to the ICON team.

## Resources

Tutorials, presentation decks, and community developed tools.
