---
title: "How to create an account"
---

There are several ways to create an account, either using SDKs or on ICONex wallet app. In this document, we will explain one by one.  

## Using T-Bears
You can create a keystore file from CLI using `tbears keystore` command. 
```bash
$ tbears keystore [keystore_file_name]
Input your keystore password : 
Retype your keystore password : 

Made keystore file successfully
```

## Using Python SDK
### Create an account
KeyWallet is an object representing an account. The code below creates a new KeyWallet instance. Internally, a private-public key pair is generated. 

```python
wallet = KeyWallet.create()
```
### Load existing account
```python
# load existing account using private key
key = bytes.fromhex(userPrivateKey)
wallet = KeyWallet.load(key)

# load existing account from keystore file
wallet = KeyWallet.load(keystoreFilePath, password)
```
### Export keystore file
```python
wallet.store(destinationFilePath, password)
```

## Using Java SDK
### Create an account
```java
KeyWallet wallet = KeyWallet.create()
```
### Load existing account
```java
// load existing account using private key
Bytes key = new Bytes(userPrivateKey)
KeyWallet wallet1 = KeyWallet.load(key);

// load existing account using keystore file
File file = new File(destinationDirectory, filename);
KeyWallet wallet2 = KeyWallet.load(password, file);
```
### Export keystore file
```java
// path to store the keystore file. keystore file name is automatically generated. 
File destinationDirectory = new File("./"); 

// keystore file password 
String password = "password_string"; 

String fileName = KeyWallet.store(wallet, password, destinationDirectory);
```

## Using ICONex
ICONex is a Chrome extention app. [Installing ICONex](https://chrome.google.com/webstore/detail/iconex/flpiciilemghbmfalicajoolhkkenfel)

### Create an account and download keystore file.
1. Click "Create Wallet". 
![img001](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex001.png)

2. Select "ICON (ICX)"
![img002](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex002.png)

3. Enter a wallet name and password. 
![img003](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex003.png)

4. Download the keystore file for backup. 
![img004](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex004.png)

5. Confirm your private key and keep it safe.
![img005](https://raw.githubusercontent.com/icon-project/documentation/master/howto/images/iconex005.png)


### Load existing account

1. Click "Load Wallet".

2. You can load your account from the keystore file ("Select wallet file") or using a private key ("Enter Private Key").


