---
layout: post
title: "RCG Sanction List Update"
date: 2024-07-14
---

## RCG Sanction List Update

Besides OFAC SDN list, there are two additional sanction lists needed for GT audit

### Consolidated Canadian Sanction List

>The Consolidated Canadian Autonomous Sanctions List includes individuals and entities subject to specific sanctions regulations made under the Special Economic Measures Act (SEMA) and the Justice for Victims of Corrupt Foreign Officials Act (JVCFOA). 

* [web site url](https://www.international.gc.ca/world-monde/international_relations-relations_internationales/sanctions/consolidated-consolide.aspx?lang=eng)
* [xml url](https://www.international.gc.ca/world-monde/assets/office_docs/international_relations-relations_internationales/sanctions/sema-lmes.xml)

* Description
   * XML includes both individual and entity by \<records> node.   
   * If \<Entity> exists, it is entity 
   * Other wise, \<LastName>, \<GivenName>  are used specify individual
   * \<aliases> are used to describe either individual or entity.
   * \<aliases> content is kind of messy format.  
      * Sometimes, they use ',', ';' to divide aliases. (tried to parse) 
      * Or even use 'or' or mixed with '/' to describe. (difficult to parse) 
      * Some use format like 'language: alias' for each alias. (try to delete the prefix language before ':')

### UN Consolidated Sanction List

* Individuals (682 individuals)
Entities and other groups (193 entities)
* XML separated by \<INDIVIDUALS> and \<ENTITIES> sections.  Each represent individual and entity records.
* In the \<INDIVIDUAL> or \<ENTITY>, there could have one or more <aliases> records.

* [Web site url](https://main.un.org/securitycouncil/en/content/un-sc-consolidated-list)
* [xml url](https://scsanctions.un.org/resources/xml/en/consolidated.xml?_gl=1*tedm2z*_ga*MjAxNjQ2NDkwMy4xNzIwOTMzMDIy*_ga_TK9BQL5X7Z*MTcyMTAzMjUwOC40LjEuMTcyMTAzMjU0Mi4wLjAuMA..)

* Mailing list Subscription

Please send your request to subscribe to sc-sanctionslists@un.org

### OFAC Sanction List

* [OFAC still the same site](https://sanctionslist.ofac.treas.gov/Home/SdnList)

### PUBLIC SAFETY CANADA

* Current list entities are `no longer used`. We are counting on Consolidated Canadian List to provide entity list.

* https://www.publicsafety.gc.ca/cnt/ntnl-scrt/cntr-trrrsm/lstd-ntts/crrnt-lstd-ntts-en.aspx