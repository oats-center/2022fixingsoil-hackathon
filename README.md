![Fixing the Soil Health Tech Stack](./docs/img/fixingsoil-logo.png)

# Fixing the Soil Health Tech Stack - 2022 Hackathon
-----------------------------------------------------

1. [Overview](#overview)  
1. [Hackathon](#hackathon-final-demo)
    1. [Teams](#teams)
    2. [Contingency Planning](#contingency-planning)
    3. [Hackathon Setup and Logistics](#hackathon-setup-and-logistics)  


# Overview
----------
The goal of this hackathon is to help "fix the soil health tech stack" by writing code together that both produces and uses soils data, thereby demonstrating interoperability and improving toolsets available to the digital soil community specifically around soil sample results and related information.

This is different than a normal hackathon where teams compete to out-do one another in particular challenges.  This hackathon is about shared engineering collaboration: we are all working together toward the final demo, and the more that works the better it is for all of us.

## Components
-------------
Tools, libraries, apps, etc. for this hackathon will likely fall into 1 of three categories:
1. **Form**: Models of soil sampling data
2. **Backfill**: Populating the standardized models with past data of various forms
3. **Function**: Apps and Analysis that can consume soil samples in the standardized models and show them in a UI or chart.

## Data
-----------
We have collected soil sampling data from several labs and made it available publicly for this hackathon.  Participants can access this data during and after the hackathon in API form at https://oats1.ecn.purdue.edu/bookmarks/soil-samples.  Details on REST API access are given below.

## Technical pre-work
---------------------
We have chosen the Modus format hosted by Ag Gateway as the main soil sampling result model for this hackathon.  We've made a JSON schema which faithfully represents the original Modus XML spec, a converter for XML into that JSON schema.  We hope to also develop, either prior to or at the hackathon, a "slim" version that is simple to navigate.

### Modus XML Schema and Nomenclature lists: 
These can be found at the Modus Bitbucket repo here: https://bitbucket.org/modus/modus-schema/src/Version-1.0/.

### New JSON-schema version of Modus
There are three components of the JSON version of Modus (to mirror the original XML structure): `global`, `modus-submit`, and `modus-result`.  `modus-result` uses definitions found in `global` and is the form in which a soils lab would communicate lab results.  Hence it will be the focus of this hackathon.

These three components are published as JSON schemas (compiled from Typescript), and associated Typescript types.  
1. `global`
Source (TS):  https://github.com/OADA/formats/blob/master/schemas/modus/v1/global.schema.cts 
JSON Schema: https://formats.openag.io/modus/v1/global.schema.json
Typescript Types (npm): `@oada/types/modus/v1/global'

2. `modus-result`
Source (TS):  https://github.com/OADA/formats/blob/master/schemas/modus/v1/modus-result.schema.cts 
JSON Schema: https://formats.openag.io/modus/v1/modus-result.schema.json
Typescript Types (npm): `@oada/types/modus/v1/modus-result'

### Universal and Command-line Javascript Tools
A monorepo of tooling is avaiable here: https://github.com/oats-center/modus.  
1. `examples`: @modusjs/examples.  Directly import-able XML and json examples of modus (great for testing).
2. `convert`: @modusjs/convert.  Javascript library to convert between formats.  Currently supports converting a Modus XML string into Modus JSON.
3. `cli`: @modusjs/cli.  Command-line tool to perform file conversions (i.e. xml to json).

## Access to Soils Data
------------------------
We hope to have API access to all the soil samples we have collected prior to this event for participants to use.  Defining this API, however, can and likely will be part of the ongoing work during the hackathon.  We will therefore provide instructions on how to access an initial simple API structure to retreive soil samples, but may add to that structure during the hackathon.


