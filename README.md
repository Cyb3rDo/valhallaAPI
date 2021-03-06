# valhallaAPI

This module allows you to interact with the Valhalla API, retrieve YARA rules in different formats, filter them and write them to disk. You can find more information about Valhalla on [our website](https://www.nextron-systems.com/yara-rule-feed/). The web interface, which doesn't have the filtering features of the Python module and the client, can be accessed [here](https://valhalla.nextron-systems.com/). 

It contains a Python module `valhallaAPI` and a Python command line API client `valhalla-cli`. 

# Python Module

The web API allows you to retrieve the subscribed rules. 

The 3 main functions of the Python module are:

- `get_rules_text()` retrieves rules as text
- `get_rules_json()` retrieves rules as JSON
- `get_rule_info()` queries that DB for info on a certain rule (e.g. hashes of samples, AV detection ratio)

The module provides functions to filter the retrieved YARA rules based on 
- tags
- score
- keywords
- supported YARA version and required YARA modules

It also allows you to retrieve a filtered rule set that fits the product that you use to apply the rules. For example, you can get a filtered rule set with rules that will run on your `FireEyeEX` appliance by filtering all rules that use feature only available in YARA versions higher than the supported `1.7.0`. 

There are 2 extra functions for special lookups in the Valhalla database (for customers only):

- `get_rule_info` retrieves rule information and all matching sample hashes
- `get_hash_info` retrieves all rules matching on a certain sha256 hash

## Demo Access

There is a demo API key that can be used for testing purposes. 

```
1111111111111111111111111111111111111111111111111111111111111111
```

It will allow you to retrieve the processed public [signature-base](https://github.com/Neo23x0/signature-base) rule set. 

The key will also allow you to query the rule info for a single rule, which is:
```
Casing_Anomaly_ByPass
```

Please note that Valhalla has protection mechanisms in place that will block your end of the Bifrost for a significant amount of time if you try foolish things.  

## Getting Started

```bash
pip install valhallaAPI
```
(make sure to use Python3.6+)

## Usage

Get a service status (does not require a valid API key)
```python
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI()
status = v.get_status()
```

Response 
```
{
  "error": "none", 
  "num_rules": 10463, 
  "status": "green", 
  "version": 2020051212
}
```

### Text Rules

Get all subscribed rules as text and save them to a file
```python
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="Your API Key")
response = v.get_rules_text()

with open('valhalla-rules.yar', 'w') as fh:
    fh.write(response)
```

Or use the DEMO API key, which allows you to retrieve all public rules
```python
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="1111111111111111111111111111111111111111111111111111111111111111")
response = v.get_rules_text()

with open('valhalla-rules.yar', 'w') as fh:
    fh.write(response)
```

Get all subscribed rules with a minimum score of 75 and save it to a file
```python
response = v.get_rules_text(score=75)
```

Get all subscribed rules that contain the keyword `Mimikatz` and save them to a file
```python
response = v.get_rules_text(search="Mimikatz")
```

Get all subscribed rules for your scan engine, which suppports YARA up to version `3.2.0` and the `pe` module, and save them to a file
```python
response = v.get_rules_text(max_version="3.2.0", modules=['pe'])
```

Get all subscribed rules for your `FireEyeEX`
```python

from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="Your API Key")
response = v.get_rules_text(product="FireEyeEX")


```
Get all subscribed rules for your `Tanium`
```python
response = v.get_rules_text(product=v.TANIUM)
```

The following products have predefined presets
```python
    FIREEYEAX = "FireEyeAX"
    FIREEYENX = "FireEyeNX"
    FIREEYEEX = "FireEyeEX"
    CARBONBLACK = "CarbonBlack"
    TANIUM = "Tanium"
    TENABLE = "Tenable"
    SYMANTECMAA = "SymantecMAA"
    GRR = "GRR"
    OSQUERY = "osquery"
    MATD3 = "McAfeeATD3"
    MATD4 = "McAfeeATD4"  
```

An example response will look like
```
/*
    VALHALLA YARA RULE SET
    Retrieved: 2019-02-25 14:54
    Generated for User: a67
    Number of Rules: 8127
    ANY REPRODUCTION OR DISTRIBUTION IS STRICTLY PROHIBITED WITHOUT THE PRIOR WRITTEN CONSENT OF NEXTRON SYSTEMS AND MAY RESULT IN LEGAL ACTION AS WELL AS THE TERMINATION OF THE CONTRACTUAL RELATIONSHIP
*/

import "pe"

rule SUSP_Katz_PDB_RID664 : EXE SUSP DEMO FILE {
   meta:
      description = "Detects suspicious PDB in file"
      author = "Florian Roth"
      reference = "Internal Research"
      date = "2019-02-04 10:32:31"
      score = 70
      customer = "demo"
      copyright = "Distribution to third parties is not permitted and will be pursued with legal measurements" 
      minimum_yara = "1.7"
      
   strings:
      $s1 = /\\Release\\[a-z]{0,8}katz.pdb/ 
      $s2 = /\\Debug\\[a-z]{0,8}katz.pdb/ 
   condition: 
      uint16 ( 0 ) == 0x5a4d and filesize < 6000KB and all of them
} 
...
```

### JSON Output

Get all subscribed rules with the `APT` tag as `JSON` and save them to a file
```python
import json
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="Your API Key")
response = v.get_rules_json(tags=['APT'])

with open('valhalla-rules.json', 'w') as fh:
    fh.write(json.dumps(response)) 
```

An example response will look like
```json
{
  "api_version": "1.0", 
  "copyright": "Nextron Systems GmbH", 
  "customer": "demo", 
  "date": "2019-03-07 10:55", 
  "legal_note": "Any reproduction or distribution is strictly prohibited without the prior written consent of Nextron Systems and may result in legal action as well as the termination of the contractual relationship", 
  "rules": [
    {
      "author": "Florian Roth", 
      "content": "rule EXP_Libre_Office_CVE_2018_16858_RIDBA9 : EXPLOIT OFFICE DEMO FILE APT {\n   meta:\n      description = \"Detects exploits addressing CVE-2018-16858 in LibreOffice - modified version\"\n      author = \"Florian Roth\"\n      reference = \"https://insert-script.blogspot.com/2019/02/libreoffice-cve-2018-16858-remote-code.html\"\n      date = \"2019-02-05 14:17:21\"\n      score = 70\n      customer = \"demo\"\n      copyright = \"Distribution to third parties is not permitted and will be pursued with legal measurements\" \n      minimum_yara = \"1.7\"\n      \n   strings:\n      $x1 = \"&#x74;&#x65;&#x6d;&#x70;&#x66;&#x69;&#x6c;&#x65;&#x70;&#x61;&#x67;&#x65;&#x72\" \n      $x2 = \"&#116;&#101;&#109;&#112;&#102;&#105;&#108;&#101;&#112;&#97;&#103;&#101;&#114;\" \n      $s1 = \"xlink:href=\\\"vnd.sun.star.script:\" ascii nocase\n      $s2 = \".py$tempfilepager\" ascii nocase\n      $s3 = \"language=Python\" ascii nocase\n   condition: \n      uint32be ( 0 ) == 0x3c3f786d and all of them or 1 of ( $x* )\n}", 
      "date": "2019-02-05 12:54:31", 
      "description": "Detects exploits addressing CVE-2018-16858 in LibreOffice - modified version", 
      "minimum_yara": "1.7", 
      "name": "EXP_Libre_Office_CVE_2018_16858_RID9B8", 
      "reference": "https://insert-script.blogspot.com/2019/02/libreoffice-cve-2018-16858-remote-code.html", 
      "required_modules": [], 
      "score": 70, 
      "tags": [
        "EXPLOIT", 
        "OFFICE", 
        "DEMO", 
        "FILE",
        "APT"
      ]
    }, 
  ...
```

### Rule Info

Get the information for rule `Casing_Anomaly_ByPass`
```python
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="Your API Key")
response = v.get_rule_info(rulename="Casing_Anomaly_ByPass")
```

Note that the rule info for `Casing_Anomaly_ByPass` is the only info that you can retrieve with the DEMO API key. 
IMPORTANT: The rule info endpoint is rate limited. You can use it for single lookups. Bulk requests lead to bans.

An example output of a rule info request will look like
```json
{
  "author": "Florian Roth", 
  "av_ratio": 16.52, 
  "av_verdicts": {
    "clean": 1, 
    "malicious": 21, 
    "suspicious": 27
  }, 
  "date": "2019-01-17 11:50:21", 
  "description": "Detects suspicious casing of bypass statement", 
  "minimum_yara": "1.7", 
  "name": "Casing_Anomaly_ByPass_RID837", 
  "reference": "Internal Research", 
  "required_modules": [], 
  "rule_matches": [
    {
      "hash": "bdde03b5b4f94ec7dbf947f3099f2009efac43b69659f788f513d3e615b98353", 
      "positives": 24, 
      "size": 319485, 
      "timestamp": "Thu, 07 Mar 2019 06:29:06 GMT", 
      "total": 56
    }, 
    {
      "hash": "646d446fb11eae76ca8b6e54306bb022431a4f20cc8cef5daa40dd6ec3537aff", 
      "positives": 3, 
      "size": 573, 
      "timestamp": "Thu, 07 Mar 2019 00:15:07 GMT", 
      "total": 57
    }
  ], 
  "score": 60, 
  "tags": [
    "SUSP", 
    "CASING"
  ] 
}
```

### Hash Info

Get the information for hash `8a883a74702f83a273e6c292c672f1144fd1cce8ee126cd90c95131e870744af`
```python
from valhallaAPI.valhalla import ValhallaAPI

v = ValhallaAPI(api_key="Your API Key")
response = v.get_rule_info(hash="8a883a74702f83a273e6c292c672f1144fd1cce8ee126cd90c95131e870744af")
```

An example output of a hash info request will look like
```json
{
    "api_version": "1.0.1",
    "results": [
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_ByPass",
            "tags": [
                "T1027",
                "SUSP",
                "CASING"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:10 GMT",
            "total": 58
        },
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_Convert_PS",
            "tags": [
                "T1027",
                "CASING",
                "SCRIPT",
                "T1064"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:12 GMT",
            "total": 58
        },
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_PowerShell",
            "tags": [
                "T1027",
                "CASING",
                "SCRIPT",
                "T1064",
                "T1086"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:13 GMT",
            "total": 58
        }
    ],
    "status": "success"
}
```

# API Client

The API client allows you to query the Web API from command line. It requires Python3.  

## Getting Started

Install Python3 and then run the following command:

```bash
pip3 install valhallaAPI
```
You should then be able to run `valhalla-cli` from command line using a Linux or macOS system. 

On Windows, do the following:
```bash
where valhalla-cli
```
Copy the full path and then run 
```bash
python C:\Python37\Scripts\valhalla-cli
```
or just download the precompiled `valhalla-cli.exe` from the latest release in the `release` section. 

## Usage

```
usage: valhalla-cli [-h] [-k apikey] [-o output-file] [--check] [--debug]
                    [-p proxy-url] [-pu proxy-user] [-pp proxy-pass]
                    [-fp product] [-fv yara-version]
                    [-fm modules [modules ...]] [-ft tags [tags ...]]
                    [-fs score] [-fq query] [--nocrypto]

Valhalla-CLI

optional arguments:
  -h, --help            show this help message and exit
  -k apikey             API KEY
  -o output-file        output file
  --check               Check subscription info and total rule count
  --debug               Debug output

=======================================================================
Proxy:
  -p proxy-url          proxy URL (e.g. https://my.proxy.net:8080)
  -pu proxy-user        proxy user
  -pp proxy-pass        proxy password

=======================================================================
Filter:
  -fp product           filter product (valid products are: FireEyeAX,
                        FireEyeNX, FireEyeEX, CarbonBlack, Tanium, Tenable,
                        SymantecMAA)
  -fv yara-version      get rules that support the given YARA version and
                        lower
  -fm modules [modules ...]
                        set a list of modules that your product supports (e.g.
                        "-fm pe hash") (setting no modules means taht all
                        modules are supported by your product)
  -ft tags [tags ...]   set a list of tags to receive (e.g. "-ft APT MAL")
  -fs score             minimum score of rules to retrieve (e.g. "-fs 75")
  -fq query             get only rules that match a certain keyword in name or
                        description (e.g. "-fq Mimikatz")
  --nocrypto            filter all rules that require YARA to be compiled with
                        crypto support (OpenSSL)
```

## Examples

Check the status of the demo user subscription
```bash
valhalla-cli --check
```

Check the status of your subscription
```bash
valhalla-cli -k YOUR-API-KEY --check
```

Get all subscribed rules and save them to `valhalla-rules.yar`
```bash
valhalla-cli -k YOUR-API-KEY
```

Get rules with score higher than 75 and save them to `valhalla-rules.yar`
```bash
valhalla-cli -k YOUR-API-KEY -fs 75
```

Get rules that work with CarbonBlack and save them to `valhalla-april-cb.yar`
```bash
valhalla-cli -k YOUR-API-KEY -fp CarbonBlack -o valhalla-april-cb.yar
```

Get rules that contain the keyword `Mimikatz` and save them to `mimikatz-rules.yar`
```bash
valhalla-cli -k YOUR-API-KEY -fq Mimikatz -o mimikatz-rules.yar
```

Get a set of rules with the highest compatibilty (lowest requirements) using the demo API key
```bash
valhalla-cli -fv 1.7
```

# Config File

Valhalla-CLI will check `~/.valhalla` as the default location for a config file. 

The config file currently contains nothing but the API key and mus look like:

```ini
[DEFAULT]
APIKEY = 786feaef202a37a8d693c57b1aeb7c8995313e358b901015c4e60033776929c3
```

# Lookups 

Valhalla-CLI has certain functions to perform lookups on its database. 

The lookups return JSON output. You can use the `-lo file` option to save the JSON results to a file.

## Hash Lookups

Hash lookup can be used to search the Valhalla database for a certain hash (SHA256 only). 

```bash
./valhalla-cli -lh 8a883a74702f83a273e6c292c672f1144fd1cce8ee126cd90c95131e870744af
```

It will return a JSON structure. 

````json
{
    "api_version": "1.0.1",
    "results": [
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_ByPass",
            "tags": [
                "T1027",
                "SUSP",
                "CASING"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:10 GMT",
            "total": 58
        },
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_Convert_PS",
            "tags": [
                "T1027",
                "CASING",
                "SCRIPT",
                "T1064"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:12 GMT",
            "total": 58
        },
        {
            "positives": 4,
            "rulename": "Casing_Anomaly_PowerShell",
            "tags": [
                "T1027",
                "CASING",
                "SCRIPT",
                "T1064",
                "T1086"
            ],
            "timestamp": "Tue, 11 Jun 2019 23:57:13 GMT",
            "total": 58
        }
    ],
    "status": "success"
}
````

## Rule Lookups

Rule lookups can be used to search the Valhalla database for a certain rule and their matches (premium feature). 

```bash
./valhalla-cli -lr Casing_Anomaly_ByPass
```

It will return a JSON structure. 

````json
{
    "author": "Florian Roth",
    "av_ratio": 18.3,
    "av_verdicts": {
        "clean": 10,
        "malicious": 59,
        "suspicious": 52
    },
    "date": "2019-01-17 11:50:21",
    "description": "Detects suspicious casing of bypass statement",
    "minimum_yara": "1.7",
    "name": "Casing_Anomaly_ByPass_RID2F47",
    "reference": "Internal Research",
    "required_modules": [],
    "rule_hash": "69b40d02020addf42cd12d3449933a3f",
    "rule_matches": [
        {
            "hash": "8a883a74702f83a273e6c292c672f1144fd1cce8ee126cd90c95131e870744af",
            "positives": 4,
            "size": 5645,
            "timestamp": "Tue, 11 Jun 2019 23:57:10 GMT",
            "total": 58
        },
        {
            "hash": "6999c997b09754fa100779af9d23a005c2a5b8944ee46175857e58e47626de65",
            "positives": 11,
            "size": 1830,
            "timestamp": "Tue, 11 Jun 2019 08:46:05 GMT",
            "total": 58
        },
    ],
    "score": 60,
    "tags": [
        "T1027",
        "SUSP",
        "CASING"
    ]
}
````

# Scores

The following list explains the scores used in the rule set

|Score|Type|Description|
|-----|----|-----------|
|1-39|Info|Low scoring rules used in our scanners (excluded from Valhalla, only used in our scanners)|
|40-59|Noteworthy|Anomaly and threat hunting rules|
|60-74|Suspicious|Rules for suspicious objects|
|75-100|Alert|Hard malicious matches|

# Important Notices

- We constantly improve old rules. They may have changed the next time you fetch the rule set. Therefore it is recommended to always fetch a full set and replace older rules with their newer versions. 
- The full rule set contains YARA rules with scores lower than 60, which are meant for threat hunting and anomaly detection use cases. 
