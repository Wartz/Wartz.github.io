---
layout: default
title: Activate Nivo 12 and Nvivo 20 for Mac with one bash script
---

Activing Nvivo 12 and 20 with an enterprise key might be easy on one computer. Or 2, or maybe even 3. But when you have to replace the activation key for an entire room, building or campus, the work rapidly spirals down into something we might call unreasonable.

I like avoiding repetitive work, so I wrote a simple Bash script to activate Nvivo for Mac en-masse.

Fortunately, Nvivo (12).app includes a silent activation command that combines a pre-filled xml file and the license key to activate the app.

```
/Applications/NVivo 12.app/Contents/MacOS/NVivo 12 \
-initialize "LICENSE" \
-activate /path/to/license_data.xml
```
The activation XML is formatted like this. I filled in the required fields with placeholder text in CAPS.

```
<?xml version="1.0" encoding="utf-8" standalone="yes"?>
<Activation>
  <Request>
    <FirstName>FIRST NAME</FirstName>
    <LastName>LAST NAME</LastName>
    <Email>EMAIL@DOMAIN</Email>
    <Phone></Phone>
    <Fax></Fax>
    <JobTitle></JobTitle>
    <Sector></Sector>
    <Industry></Industry>
    <Role></Role>
    <Department></Department>
    <Organization></Organization>
    <City>CITY</City>
    <Country>NATION</Country>
    <State>STATE</State>
  </Request>
</Activation>
```
I wanted a single use script for both Nvivo 12 and 20, where I could specify which app I was installing an activating with a flag.

Full script is my github.
[https://github.com/Wartz/automation-tools/blob/main/nvivo-activate.sh](https://github.com/Wartz/automation-tools/blob/main/nvivo-activate.sh)