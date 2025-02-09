---
title: Access privileges
date: 2024-04-07
tags:
  - technical
  - snowflake
---

| **Services**                         | **Access Privileges**                                                                                                                               |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------------------------------- |
| Search Optimization services         | OWNERSHIP privilege on the table  <br>ADD SEARCH OPTIMIZATION privilege on the schema that contains the table                                       |
| to query a table                     | USAGE on db  <br>USAGE on schema  <br>SELECT on table                                                                                               |
| Creating and managing external table | Database - USAGE  <br>Schema - USAGE, CREATE STAGE (if creating a new stage), CREATE EXTERNAL TABLE  <br>Stage (if using an existing stage) - USAGE |
| Data listing                         | CREATE DATABASE, IMPORT SHARE                                                                                                                       |
| Data Exchange                        | IMPORTED PRIVILEGES                                                                                                                                 |
| Shared DB                            | IMPORTED PRIVILEGES                                                                                                                                 |
|                                      |                                                                                                                                                     |
|                                      |                                                                                                                                                     |
