---
title: "How to change network in ICONex Chrome extension"
---

When you develop a DApp, you will spend most of your time to test your smart contract on the testnet. ICONex chrome app is a good utility helping DApp development, as you can invoke smart contract functions from the web app without writing a single line of code, as well as sending ICX to test accounts. However, when you install the ICONex chrome app, the default configuration is connected to the mainnet, so, you can not send transactions to the accounts or smart contracts on the testnet. Thus, in this document, we will explain how to connect your ICONex chrome app to the testnet.  

- Open the Chrome DevTools by pressing F12, then go to the **Application** tab. In the **Storage** section, expand **Local Storage**. 
- Add a new key/value pair, **isDev/true**, by clicking on the empty row at the bottom of the table.

![](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex-isdev.png)

- Reload your wallet, then you will see the menu in the bottom. Click the **ICX (SERVER)** button to open the dropup list of the available networks. You can choose predefined one, or manually set a custom node. 

![](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex-network.png)

 


