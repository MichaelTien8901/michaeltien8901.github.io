---
layout: post
title: "Conda Install 32bits Python"
tags: [Python, Access]
date: 2022-03-26
---

## Can't use Access ODBC Driver for Access Database file(*.mdb)

   Even we use "ODBC Data Sources(32bits)" program to find the driver as in the following code.

```python
import pyodbc
import datetime

DATABASE_PATH='d:\Projects\doc\test.mdb'
conn = pyodbc.connect(r'Driver={Microsoft Access Driver (*.mdb, *.accdb)};DBQ=' + DATABASE_PATH)
cursor = conn.cursor()
```

The reason is that we are using 32 bits ODBC that need 32 bits Python.

## Install Python 32 bits

```
set CONDA_FORCE_32BIT=1
conda create -n py3_32bit python=3.8
enable py3_32bit
```

* Just remember to set CONDA_FORCE_32BIT= (set empty) if you are going to use the root environment 64bit

## Install pyodbc

   In order to use mdb access database file, need to install pyodbc.

```
conda install pyodbc
```

## Install ipykernel

When run the previous python code in `VS code for Jupyter Notebook`, error message that we need to install ipykernel

```
conda install ipykernel
```


