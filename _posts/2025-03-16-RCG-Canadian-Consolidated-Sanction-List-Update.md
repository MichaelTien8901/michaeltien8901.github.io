---
layout: post
title: "RCG Canadian Consolidated Sanction List Update"
date: 2025-03-16
---

## RCG Canadian Consolidated Sanction List Update

* XML format have changed.  With additional possible property names used.  

```XML
<Country>Ukraine</Country>
<EntityOrShip>Chernomorneftegaz</EntityOrShip>
<Schedule>Part 2</Schedule>
<Item>1</Item>
<DateOfListing>2014-04-12</DateOfListing>
```

* Update the parsing code for Canadian Consolidated Sanction List.
   1. Instead of hardcoded property names, use a string list to list all entity property names, except defatult "Entity".  
   2. Add rules to prevent individual names with empty values.

