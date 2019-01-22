---
title: "Reference Guide Template"
excerpt: ""
---

Document always should start with a compelling overview. No headings for overview.
In this opening pharagraph, state what this document is about, what it will explain. 
An overview helps potential readers to determine quickly if a particular how-to 
matches their interests or needs.

## Heading 
Be accurate and complete. Give usage samples. Show example value for each data type. 
Cover possible valid and invalid cases, enumerate error codes.  

Console example.
```bash
$ tbears keystore [keystore_file_name]
Input your keystore password : 
Retype your keystore password : 

Made keystore file successfully
```

Python code example.
```python
# load existing account using private key
key = bytes.fromhex(userPrivateKey)
wallet = KeyWallet.load(key)

# load existing account from keystore file
wallet = KeyWallet.load(keystoreFilePath, password)
```

Java code example.
```java
// load existing account using private key
Bytes key = new Bytes(userPrivateKey)
KeyWallet wallet1 = KeyWallet.load(key);

// load existing account using keystore file
File file = new File(destinationDirectory, filename);
KeyWallet wallet2 = KeyWallet.load(password, file);

```


- step 1
  - step 1-1
- step 2
- step 3
  - step 3-1

When using a bullet list do not insert a new line between items. 

Do this. 
```
- step 1
  - step 1-1
- step 2

Add one line spacing at the end of list.
```

Don't do this. This will break the style on the readme.io.
```
- step 1

  - step 1-1

- step 2
```

### Subsection Header 3

For the header, every words except some little words (like, a, the, of, for, which .. ) 
should start with an uppercase letter 

#### Header 4 

Do not recommend using header size below 4. 

1. Numbered list
2. Numbered list
  - Test
3. Numbered list



