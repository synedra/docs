<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/font-awesome/4.6.1/css/font-awesome.min.css">
<link rel="stylesheet" type="text/css" href="../../../../assets/stylesheets/formbase.min.css">

<link rel="stylesheet" type="text/css" href="https://unpkg.com/swagger-ui-dist@3.25.1/swagger-ui.css">
<script src="https://unpkg.com/swagger-ui-dist@3.25.1/swagger-ui-standalone-preset.js"></script>
<script src="https://unpkg.com/swagger-ui-dist@3.25.1/swagger-ui-bundle.js"></script>
<script src="../../../../assets/javascripts/swagger-sandbox.js"></script>

The **Document API** is an HTTP REST API and part of the open source [Stargate.io](https://stargate.io/). The idea is to provide an abstraction on top of Apache Cassandra™ to allow **document-oriented** access patterns.

<img src="../../../../img/stargate-api-doc/architecture.png" />

- A `namespace` (replacement for keyspace) will hold multiple `collections` (not tables) to store `Documents`

- You interact with the database through `JSON documents` and no validation (sometimes called `_schemaless_` but a better term would be validationless).

- Each documents has a unique identifier within the collection. Each insert is an upsert.

- You can query on any field (_thanks to out of the box support for the secondary index `SAI`_)

```mermaid
  graph LR
    DB(Database) -->|1...n|NS(Namespaces)
    NS -->|1..n|COL(Collections)
    COL -->|1..n|DOC(Documents)
    DOC -->|1..49 Nested docs|DOC
```

???+ abstract "How is the data stored in Cassandra?"

      The JSON documents are stored using an internal data model. The table schema is generic as is each collection. The algorithm used to transform the document is called **_document shredding_**. The schema is optimized for searches but also to limit tombstones on edits and deletes.

      ```sql
      create table <collection_name> (
        key       text,
        p0        text,
        ...
        p[N]       text,
        bool_value boolean,
        txt_value  text,
        dbl_value  double,
        leaf       text
      )
      ```

      A JSON like `{"a": { "b": 1 }, "c": 2}` will be stored like

      | key | p0 | p1 | dbl*value |
      |:--------------:|:--------------:|:-----------|:-----------|
      | {docid} | `a` | `b` | `1` |
      | {docid} | `c` | \_null* | `2` |

      This also works with arrays `{"a": { "b": 1 }, "c": [{"d": 2}]}`

      |   key   | p0  | p1    | p2     | dbl_value |
      | :-----: | :-: | :---- | :----- | :-------- |
      | {docid} | `a` | `b`   | _null_ | `1`       |
      | {docid} | `c` | `[0]` | `d`    | `2`       |

## Prerequesites

In order to use the **`Document API`** for Astra DB in your application, the following prerequisites must be met.

- An **Astra account**, use the [tutorial](http://astra.datastax.com/) to create yours
- A running **Astra Database**, use the [tutorial](https://github.com/datastaxdevs/awesome-astra/wiki/Create-an-AstraDB-Instance) to create one
- An **authentication token**, use the [tutorial](https://github.com/datastaxdevs/awesome-astra/wiki/Create-an-Astra-Token) to create one
- A client, framework or tool to execute **Http Requests**. This page will provide you `SWAGGER` and `POSTMAN`

## Database Selector

<fieldset>
<legend>Astra DB Setup</legend>
<label class="label" for="astra_token"><i class="fa fa-key"></i> &nbsp;Authentication token&nbsp;<sup>*</sup></label>
<span id="astra_token_errors" style="color:red;font-style:italic;"></span>
<br/>
<input class="input" id="astra_token" name="astra_token" type="text" placeholder="AstraCS:...." style="width:70%">

<!-- Waiting for the Devops API to Allow CORS
<input type="submit"
       class="md-button button-primary float-right" value="Lookup Databases"
       onclick="dbSelectorListDatabases(document.getElementById('astra_token').value)" />
-->

<div id="block_astra_db">
  <label class="label" for="astra_db"><i class="fa fa-database"></i> &nbsp;Database identifier&nbsp;<sup>*</sup> <a href="/pages/astra/faq/#where-should-i-find-a-database-identifier">(Where find it ?)</a></label>
  <span id="astra_db_errors" style="color:red;font-style:italic;"></span>
  <br/>
  <input class="input" id="astra_db" name="astra_token" type="text" placeholder="Your Database id" style="width:70%">
</div>

<div id="block_astra_region">
  <label class="label" for="astra_region"><i class="fa fa-map"></i> &nbsp;Database Region&nbsp;<sup>*</sup>  <a href="/pages/astra/faq/#where-should-i-find-a-database-region-name">(Where find it ?)</a></label>
   <span id="astra_region_errors" style="color:red;font-style:italic;"></span>
  <br/>
  <select class="select" id="astra_region" 
    name="astra_region" style="width:70%" 
    onchange="dbSelectorShowKeyspaces(
      document.getElementById('astra_token').value, 
      document.getElementById('astra_db').value, 
      document.getElementById('astra_region').value)">
    <option selected disabled>Pick your region</option>
    <optgroup label="Google Cloud Platform">
      <option value="asia-south1">(GCP) asia-south1</option>
      <option value="europe-west1">(GCP) europe-west1</option>
      <option value="europe-west2">(GCP) europe-west2 </option>
      <option value="northamerica-northeast1">(GCP) northamerica-northeast1</option>
      <option value="southamerica-east1">(GCP) southamerica-east1</option>
      <option value="us-central1">(GCP) us-central1</option>
      <option value="us-east1">(GCP) us-east1</option>
      <option value="us-east4">(GCP) us-east4</option>
      <option value="us-west1">(GCP) us-west1</option>
    </optgroup>
    <optgroup label="AWS">
      <option value="ap-southeast-1">(AWS) ap-southeast-1</option>
      <option value="eu-central-1">(AWS) eu-central-1</option>
      <option value="eu-west-1">(AWS) eu-west-1</option>
      <option value="us-east-1">(AWS) us-east-1</option>
      <option value="us-east-2">(AWS) us-east-2</option>
      <option value="us-west-2">(AWS) us-west-2</option>
    </optgroup>
    <optgroup label="Azure">
      <option value="northeurope">(Azure) northeurope</option>
      <option value="westeurope">(Azure) westeurope</option>
      <option value="eastus">(Azure) eastus</option>
      <option value="eastus2">(Azure) eastus2</option>
      <option value="southcentralus">(Azure) southcentralus</option>
      <option value="westus2">(Azure) westus2</option>
      <option value="canadacentral">(Azure) canadacentral</option>
      <option value="brazilsouth">(Azure) brazilsouth</option>
      <option value="centralindia">(Azure) centralindia</option>
      <option value="australiaeast">(Azure) australiaeast</option>
    </optgroup>
  </select>
</div>

<div id="dbselector_errors" style="color:red;font-style:italic;"></div>

<div id="block_astra_namespace" >
</div>

</fieldset>

## Swagger Sandbox

<div id="swagger-ui"></div>

<script>

function setupSwagger() {
  window.ui = SwaggerUIBundle({
    url: "../swagger-api-document.json",
    dom_id: '#swagger-ui',
    presets: [
      SwaggerUIBundle.presets.apis,
      SwaggerUIStandalonePreset
    ],
    plugins: [
      UrlMutatorPlugin
    ],
    layout: "StandaloneLayout",
    onComplete: () => {
       dbSelectorBuildStargateEndpoint('ASTRA_DB_ID', 'ASTRA_DB_REGION')
    } 
  });
  document.querySelector(".topbar").hidden=true;
  // Add the populate field function Hook.
  setTimeout(hookSwagger, 100);
}

window.onload = setupSwagger;
  
</script>

## Postman

**Prerequisites [Development Environment]**

- You should install **[Postman](https://www.postman.com/downloads/)** to import the sample collections that we have provided.

**Setup Postman**

- Import the configuration File `Astra_Document_Api_Configuration.json` in postman. In the menu locate `File > Import` and drag the file in the box.

![import-doc](https://github.com/datastaxdevs/awesome-astra/blob/main/postman/docapi-conf-import.png?raw=true)

- Edit the values for you db:

| Parameter Name | parameter value                       | Description                                                                                                                                                                                                                                       |
| :------------: | :------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
|     token      | `AstraCS:....`                        | _When you generate a new token it is the third field. Make sure you add enough privileges to use the APis, Database Administrator is a good choice to develop_                                                                                    |
|       db       | `00000000-0000-0000-0000-00000000000` | _Unique identifier of your DB, [you find on the main dashboard](https://github.com/datastaxdevs/awesome-astra/wiki/Astra-FAQ#where-should-i-find-a-database-identifier-)_                                                                         |
|     region     | `us-east1`                            | _region name, [you find on the datanase dashboard](https://github.com/datastaxdevs/awesome-astra/wiki/Astra-FAQ#where-should-i-find-a-database-region-name-)_                                                                                     |
|   namespace    | `demo`                                | _Namespaces are the same as keyspaces. They are created with the database or added from the database dashboard: [How to create a keyspace](https://github.com/datastaxdevs/awesome-astra/wiki/Astra-FAQ#how-to-create-a-namespace-or-keyspace-)]_ |
|   collection   | `person`                              | _Collection name (like table) to store one type of documents._                                                                                                                                                                                    |

- this is what it is looks like

![import-doc](https://github.com/datastaxdevs/awesome-astra/blob/main/postman/docapi-conf-edit.png?raw=true)

- Import the Document Api Collection `Astra_Document_Api.json` in postman. Same as before `File > Menu`

![import-doc](https://github.com/datastaxdevs/awesome-astra/blob/main/postman/docapi-import.png?raw=true)

- That's it! You have now access to a few dozens operations for `namespace`, `collections` and `documents`

![import-doc](https://github.com/datastaxdevs/awesome-astra/blob/main/postman/docapi-resources.png?raw=true)

## Working with CURL

- You should have **curl** commands available but, if not, you can install from [here](https://curl.se/download.html) or

```bash
curl --version
```

## Operations

!!! warning "Known Limitations"

      - As of today there is **no aggregation or sorting** available in the Document Api.

      - Queries are paged with a **pagesize of `3` records by default** and you can increase up to a maximum of `20` records. Otherwise, the payload would be too large.

### ‣ List Namespaces

!!! example "`GET /v2/namespaces/`"

      **Definition:**

      This operation lists the namespaces inside of a Cassandra database. For each namespace, it will list the datacenters where it resides and the number of replicas.

      **Sample Curl**

      ```bash
      curl -X GET https://{ASTRA_DB_ID}-{ASTRA_DB_REGION}}\
      .apps.astra.datastax.com/api/rest/v2/namespaces/ \
        -H 'accept: application/json' \
        -H 'X-Cassandra-Token: {TOKEN}}'
      ```

      **Sample Output**

      ```json

      ```

## Extra Resources

!!! abstract "Reference Documentation"

    <ol>
        <li><a href="https://stargate.io/2020/10/19/the-stargate-cassandra-documents-api.html">Document API reference Blogpost</a>
        <li><a href="https://stargate.io/2021/04/05/the-stargate-documents-api-storage-mechanisms-search-filters-and-performance-improvements.html">Design Improvements in 2021</a>
        <li><a href="https://stargate.io/docs/stargate/1.0/quickstart/quick_start-document.html">QuickStart</a>
    </ol>