---
title: "JSON Standard for P-Rep Detailed Information"
excerpt: "The source code is found on GitHub at https://github.com/icon-project/"
---

This document is a guideline detailing how to register, unregister, and set a Public Representative (“P-Rep”) node on the MainNet using preptools. P-Reps are the consensus nodes that produce, verify blocks and participate in network policy decisions on the ICON Network.

## Intended Audience
We recommend all P-Rep candidates to go through this guideline and participate in the block generation test.

## JSON Standard for Public Representative Detailed Information
This is the JSON standard for detailed information about the P-Rep. P-Rep can submit the url of detailed information via the `registerPRep` and `setPRep` action on the ICON Blockchain. We strongly recommend that you register this information.

```json
{
    "representative": {
        "logo": {
            "logo_256": "https://icon.foundation/img/img-256.png",
            "logo_1024": "https://icon.foundation/img/img-1024.png",
            "logo_svg": "https://icon.foundation/img/img-logo.svg"
        },
        "media": {
            "steemit": "",
            "twitter": "",
            "youtube": "",
            "facebook": "",
            "github": "",
            "reddit": "",
            "keybase": "",
            "telegram": "",
            "wechat": ""
        }
    },
    "server": {
        "location": {
            "country": "",
            "city": ""
        },
        "server_type": "",
        "api_endpoint": ""
    }
}
```

- representative: Basic information of Public Representative
  - logo: Logo images of P-Rep
    -  logo_256: image 256x256px
    -  logo_1024: image 1024x1024px
    -  logo_sgv: image svg
  - media: URL and username of social media
    -  steemit: Steemit URL
    -  twitter: Twitter URL
    -  youtube: Youtube URL
    -  Facebook: Facebook URL
    -  github: Github URL
    -  reddit: Raddit URL
    -  keybase: Username
    -  telegram: Username
    -  wechat: Username
- server: Server information of Public Representative
  - location: Server location
    -  country: Node country code in accordance to [ISO 3166-1 alpha-2](https://en.wikipedia.org/wiki/ISO_3166-1_alpha-2)
    -  city: Node city in human readable format
  - server_type: Type of server ‘cloud, on-premise, hybrid’
  - api_endpoint: HTTP endpoint http://host:port

### How to use
Create a JSON file and upload it to your domain server. When you call the `registerPRep` or `setPRep` function, input the url of this file into the `details` field.

### Example
```json
{
  "representative": {
    "logo": {
      "logo_256": "https://icon.foundation/img/img-256.png",
      "logo_1024": "https://icon.foundation/img/img-1024.png",
      "logo_svg": "https://icon.foundation/img/img-logo.svg"
    },
    "media": {
      "steemit": "https://steemit.com/@iconproject",
      "twitter": "https://twitter.com/iconproject",
      "youtube": "https://www.youtube.com/channel/iconproject",
      "facebook": "https://www.facebook.com/iconproject",
      "github": "https://github.com/iconproject",
      "reddit": "https://www.reddit.com/user/iconproject",
      "keybase": "@iconproject",
      "telegram": "@iconproject",
      "wechat": "@iconproject"
    },
  },
  "server": {
    "location": {
      "country": "KOR",
      "city": "Seoul"
    },
  "server_type": "on-premise",
  "api_endpoint": ""
  },
}
```

## References
- [Set P-Rep using preptools](https://github.com/icon-project/preptools#preptools-setprep)


## License
This project follows the Apache 2.0 License. Please refer to [LICENSE](https://www.apache.org/licenses/LICENSE-2.0) for details.
