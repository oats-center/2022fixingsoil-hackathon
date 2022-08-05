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
We have collected soil sampling data from several labs and made it available publicly for this hackathon.  Participants can access this data in API form at https://oats1.ecn.purdue.edu/bookmarks/soil-samples.  Details on REST API access are given below.

## Technical pre-work
---------------------
We have chosen the Modus format hosted by Ag Gateway as the main soil sampling result model for this hackathon.  We've made a JSON schema which faithfully represents the originla Modus XML spec, a converter for XML into that JSON schema, and a "slim" version JSON schema with converter.  

### Modus XML Schema and Nomenclature lists: 
These can be found at the Modus Bitbucket repo here: https://bitbucket.org/modus/modus-schema/src/Version-1.0/.

### New JSON-schema version of Modus
This is defined in the handy OADA Formats repo, which automatically builds and publishes Typescript typings for JSON schemas.
| Global Definitions   | Modus Result  |
|----------------------|---------------|
| Schema | https://github.com/OADA/formats/blob/master/schemas/modus/v1/global.schema.cts | https://github.com/OADA/formats/blob/master/schemas/modus/v1/modus-result.schema.cts |



https://formats.openag.io/modus/v1/global.schema.json | @oada/types/modus/v1/global.js |
| Lab Results       |  https://github.com/OADA/formats/blob/master/schemas/modus/v1/modus-result.schema.cts | https://formats.openag.io/modus/v1/modus-result.schema.json | @oada/types/modus/v1/modus-result.js |
