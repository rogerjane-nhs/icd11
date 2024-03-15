# Investigation into ICD-11 API

Top API page: https://icd.who.int/icdapi

Gaining an API key is simply a case of registering on the WHO page, then visiting the page above and clicking 'client keys'.

API documentation is here: https://icd.who.int/docs/icd-api/APIDoc-Version2/

Swagger interface is here: https://id.who.int/swagger/index.html

Examples for access can be found here: https://github.com/icd-api

Beta documentation on the content model is here: https://icd.who.int/icdapi/docs/ContentModelGuide.pdf (134 pages of it)

Mapping tables come from the UI at https://icd.who.int/browse/2024-01/mms/en - hover on 'info' then icd10/icd11 mapping tables - https://icdcdn.who.int/static/releasefiles/2024-01/mapping.zip

## Setup

You'll need a config file (icd11.cfg) in the same file as the icd11 binary.

To make one, go to https://icd.who.int/icdapi and click on 'register' under 'API Access'.  Once this is done, click 'View API Access key(s)'
on the same page (or go to https://icd.who.int/icdapi/Account/AccessKey).  Copy the two lines
and paste them into a file called icd11.cfg.  It should look something like:

```
ClientId: 12345678-abcd-4099-93d6-08fcef492011_87654321-1234-4dbb-ba2f-96e7c84e27c3
ClientSecret: k234dhliuy4gr1vgct2jhg345kbvdk23hb4v431h34k=
```


## Oddities

```
RogMacbook-2:icd11 rog$ icd11 map F00.0
ICD10 F00.0     Dementia in Alzheimer disease with early onset
ICD11 6D80.0    Dementia due to Alzheimer disease with early onset
RogMacbook-2:icd11 rog$ icd11 map G30.0
ICD10 G30.0     Alzheimer disease with early onset (age up to 65)
ICD11 6D80.Z    Dementia due to Alzheimer disease, onset unknown or unspecified
```

## Email sent:

```
There seem to be two parts to this.
 
### The API.
 
The API talks almost entirely in terms of ‘entities’, which use URI identifiers (e.g. http://id.who.int/icd/release/11/2024-01/mms/795022044/unspecified) – the interesting parts of this are:
 
                mms                     Mortality and Morbidity Statistics (ICD-11)
795022944        Numeric identifier (for “Dementia due to Alzheimer disease”)
unspecified       A ‘residual’ specifier  (could also be ‘other’ or, more usually, omitted)
 
Given the required authentication, a call to the above URI will return lots of information regarding the entity including alphanumeric code, parents, children, accepted post coordination, title, synonyms etc.  Notably the ‘code’ is ‘nullable’, implying that there isn’t always an ICD11 code to represent the entity.  There is, however, a call that gives the URI given the ICD11 code (including possible post coordination).
 
I believe the scope of a ‘foundation entity’ is wider than ICD-11, hence the optionality of an ICD-11 code in the response above.
 
### The mapping files.
 
The other part is a set of mapping files.  These are not available (that I could see) via the API and instead need to be downloaded locally for use.  There are five of these, each in two formats (plain text and excel):
 
ICD10 to ICD11 (One category)
ICD10 to ICD11 (Multiple categories)
ICD11 to ICD10 (One category)
Foundation ICD10 to ICD11 (One category)
Foundation ICD11 to ICD10 (One category)
 
I couldn’t locate documentation on these (perhaps I could have looked harder) but I did investigate the first two and they seem to do what they imply in the names.
 
I’ve written a Python script to exercise both the API and mapping files including mapping ICD10 –> ICD 11, finding an entity given an ICD11 code, showing the details of an entity etc.  It could certainly be used as a much fuller example of accessing the API than the excuse for an example that WHO provide.  The script could be expanded for further functionality quite easily.
```
