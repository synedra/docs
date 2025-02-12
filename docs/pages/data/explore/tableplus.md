---
title: "TablePlus"
description: "TablePlus is a modern, native tool with elegant UI that allows you to simultaneously manage multiple databases such as MySQL, PostgreSQL, SQLite, Microsoft SQL Server and more."
tags: "jdbc, data management, ide"
icon: "https://awesome-astra.github.io/docs/img/tableplus/tableplus.svg"
developer_title: "TablePlus"
developer_url: "https://tableplus.com/download"
links:
- title: "Download TablePlus"
  url: "https://tableplus.com/"
- title: "TablePlus Linux Installation"
  url: "https://tableplus.com/blog/2019/10/tableplus-linux-installation.html"
---
<div class="nosurface" markdown="1">

<img src="https://awesome-astra.github.io/docs/img/tableplus/download.png" />


- _This article includes information that was originally written by **Erick Ramirez** on [DataStax Community](https://community.datastax.com/articles/12299/how-to-connect-to-astra-db-from-tableplus.html)_
</div>

## Overview

TablePlus is a modern, native tool with elegant UI that allows you to simultaneously manage multiple databases such as MySQL, PostgreSQL, SQLite, Microsoft SQL Server and more.

- <span class="nosurface">ℹ️ </span>[Introduction to TablePlus](https://docs.tableplus.com/getting-started)
- <span class="nosurface">📥 </span>[TablePlus Download Link](https://docs.tableplus.com/#download-and-install)

## Prerequisites

<ul class="prerequisites">
  <li class="nosurface">You should have an <a href="https://astra.dev/3B7HcYo">Astra account</a></li>
  <li class="nosurface">You should <a href="https://awesome-astra.github.io/docs/pages/astra/create-instance/">Create an Astra Database</a></li>
  <li class="nosurface">You should <a href="https://awesome-astra.github.io/docs/pages/astra/create-token/">Have an Astra Token</a></li>
  <li class="nosurface">You should <a href="https://awesome-astra.github.io/docs/pages/astra/download-scb/">Download your Secure bundle</a></li>
</ul>

This article assumes you have a running installation of Tableplus on your laptop or PC. It was written for the MacOS version but it should also work for the Windows version.

## Installation and Setup

!!! note "Note" 
  For simplicity, the secure connect bundle has been placed in `/path/to/scb`

### <span class="nosurface">✅ Step 1:</span> DB Information

On your laptop or PC where Tableplus is installed, unpack your secure bundle. For example:

```
$ cd /path/to/scb
$ unzip secure-connect-getvaxxed.zip
```

Here is an example file listing after unpacking the bundle:

```bash
/
  path/
    to/
      scb/
        ca.crt
        cert
        cert.pfx
        config.json
        cqlshrc
        identity.jks
        key
        trustStore.jks
```

Obtain information about your database from the config.json file. Here is an example:

```bash
{
  "host": "<YOUR_ENDPOINT>.db.astra.datastax.com",
  "port": 98765,
  "cql_port": 34567,
  "keyspace": "<KEYSPACE_NAME>",
  "localDC": "us-west-2",
  "caCertLocation": "./ca.crt",
  "keyLocation": "./key",
  "certLocation": "./cert",
  ...
}
```

We will use this information to configure Astra DB as the data source in Tableplus.

### <span class="nosurface">✅ Step 2:</span> New Connection

1. In Tableplus, create a new connection and select **Cassandra** as the target database.

2. In the **Host** and **Port** fields, use the `host` and `cql_port` values in the `config.json` above.

3. In the **User** and **Password** fields, use the client ID and client secret from the token you created in the **Prerequisites** section of this article.

4. In the **Keyspace** field, use the `keyspace` values in the `config.json` above.

5. Choose `SSL VERIFY NONE` for the **SSL mode.**

6. For SSL keys, select the secure bundle files:
!!! note "Secure Bundle Files"
    - `key` for Private Key (leave the password blank when prompted)
    - `cert` for Cert
    - `ca.crt` for Trusted Cert


Here's an example of what the **Cassandra Connection** dialog box should look like:

<img src="https://awesome-astra.github.io/docs/img/tableplus/tableplus_connection_page_updated.png" height="350px" />
<br /><br />

### <span class="nosurface">✅ Step 3:</span> Final Test

Connect to your Astra DB. If the connection was successful, you should be able to see all the tables on the left-hand side of the UI.

Here's an example output:

<img src="https://awesome-astra.github.io/docs/img/tableplus/2384-tableplus-astra-connected.png" height="350px" />
