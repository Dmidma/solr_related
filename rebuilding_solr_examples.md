* Solr is a complex product:
    - wide
    - deep
    - many examples


* Going to see all OOTB examples
* Make a small one from scrath

* How many OOTB examples?

...

#### techproducts example

* `solr.home`: `example/techproducts/solr`  
    All the config, data, rest of the files will be.
* restart with: `bin/solr start -s example/techproducts/solr`
* core: `example/techproducts/solr/techproducts`
* source configuration: `server/solr/configset/sample_techproducts_configs`  
    Not actually a configest (copy).
* Data: 14 files of products, money, utf8 tests
* rebuilt:
```
rm -rf example/techproducts
bin/post -c techproducts example/exampledocs/*.xml
```

#### schemaless example:

* `solr.home`: `example/schemaless/solr`  
* restart with: `bin/solr start -s example/schemaless/solr`
* core: `example/schemaless/solr/gettingstarted`
* source configuration: `server/solr/configset/data_driven_schema_configs`  
    Not actually a configest (copy).

> This is the config you get when you are not using config when creating a core.
```
bin/solr create -c newcore
```

* No data but can take anything:
```
bin/post -c <name> example/exampledocs/*.xml
```

* Solr will try to guess what you mean when you throw data at it.
* Auto-guess filed type based on **first** content occurrence.
* Create explicit field definitions (always mutlivalued):
    * booleans
    * dates
    * numbers
    * strings
    
    * Can be configured in `solrconfig.xml`

* Rewrites managed-schema (comment begone!)
* Makes search work with `<copyField source="*" dest="_test_"/>`

##### techproducts vs schemaless:

 types |techproducts | schemaless
---------|------------------|------------
Strings | "name": "Test with Some GB18030 encoded characters", |"name": ["Test with Some GB18030 encoded characters"], 
Numbers | "price":"0.0", "price\_c":"0.0,USD", | "price":[0.0],
Boleans | "inStock":true, | "inStock":[true],



#### cloud example:

* Highly configurable (unless using `-noprompt`; default config: `data_driven_schema_configs`)
* `solr.home`: `example/cloud/nodeX/solr`
* Source configuration is a choice:
    * `basic_configs`
    * `data_driven_schema_configs`
    * `sample_techproducts_configs`
* rebuilt:
```
bin/solr stop -all
rm -rf example/cloud
```


#### dih example(s):

* Data import handler - legacy, but still kicking
* `solr.home`: `example/example-DIH/solr`
* Have 5 different cores:
    - db - database import (`example/example-DIH/hsqldb/ex.*)
    - solr - import from another solr core (configured for db core)
    - mail - import from IMAP (some configuration)
    - tika - import rich-content (`example/exampledocs/solr-word.pdf`)
    - rss - external XML feed (broken!)

* Cannot be rebuilt - only emptied:
```
bin/post -c db -type 'application/json' -d 'delete: {query:"*:*"}}'
```


#### bin/solr start?

* `solr.home`: `server/solr`
* No initial collection/cores, have to create explicitly:
    - `bin/solr create_core -h` for details:
        ` bin/solr create_core -c <corename> -d <name or path>`

    - With core Admin UI for non-SolrCloud:
        `http://localhost:8989/solr/admin/cores?action=CREATE&...`

    - With collection API for SolrCloud:
        `http://localhost:8989/solr/admin/collections?action=CREATE&...`


#### basic\_configs configuration

* Available for cloud example and explicit creation
* Schemaless mode is configured, not enabled
* "Minimal Solr Configuration"
    - managed-schema: 1005 lines
    - solrconfig.xml: 1484 lines


#### files example:

* Specifically tuned for file indexing
    - Augmented schemaless mode with language, content-type guessing
    - Custom /browse end-point

* Source configuration: `example/files/conf`
* Setup instructions: `example/files/README.txt`
* Bring your own data


#### films example:

* Schemaless (based on data_driven_schema_configs)
    - Uses Schema API to add custom fields
    - Uses schemales for rest of fields

* 1100 film records
* Uses:
    * velocity /browse
    * Schema API
    * Request parameters API (params.json)
* Setup instructions: `example/files/README.txt`



#### Minimal Example: 

* managed-schema (file in the config directory with solrconfig.xml):
```
<schema name="demo" version="1.6">

    <dynamicField name="*" type="string" indexed="true" stored="true" multiValued="true"/>
    
    <field name="text" type="test_basic" indexed="true" stored="false" multiValued="true"/>

    <copyField source="*" dest="text"/>

    <fieldType name="string" class="solr.StrField"/>

    <fieldType name="text_basic" class="solr.TextField">
        <analyzer>
            <tokenizer class="solr.LowerCaseTokenizeFactory"/>
        </analyzer>
    </fieldType>
</schema>
```

* Copy every thing from that * to the **text**.

* The `lucenMatchVersion` is must have, all else is optional.
* If you are running **SolrCloud**, there is more couple of things.


* solrconfig.xml:
```
<config>
    <luceneMatchVersion>6.2.0</luceneMatchVersion>

    <requestHandler name="/select" class="sorl.SearchHandler">
        <lst name="defaults">
            <str name="df">text</str>
        </lst>
    </requestHandler>
</config>
```

* Load it:
```
bin/solr start
bin/solr create -c demo -d ../demo-config/
bin/post -c demo example/exampledocs/*.xml
```

* Test it:
```
http -b :8983/solr/demo/select q==*:* rows==0
http -b :8983/solr/demo/select q==ipod rows==1 facet==true facet.field==cat facet.mincount==1
```
You can use HTTPie (HTTP CLI)


* Some things will not work:
    - No `uniqueKey` - no way to update documents
    - No `_version_` - no SolrCloud
    - Everything is multivalued - no sorting
    - copyField * -> text, mo meaningful relevancy, specialized analyzer chain processing



#### Deconstructing films example:

```
bin/solr create -c films
```

* Index 1100 records from
    - (Solr) XML,
    - (generic) JSON (doc),
    - CSV format
same data from three different way.

* README.txt tells you what you should do.


> **params.json** is a config file on top of the **solrconfig.xml**.


##### Deconstructing - just straight tags:

- managed-schema lost comments durig construction
- remove comments from solrconfig.xml `xml ed -L -d "//comment()" solrconfig.xml

> The number of lines dropes from 1400 to 250.

##### Deconstructing - what to clean:

- (explicit) fields: 8
- dynamic fields: 73
    * `xml set -t m "//dynamicField" -v @name -n managed-schema | wc -l`
- types: 71
- copyFields: 1

##### Deconstructing - dynamic fields:
 
- Used dynamic fie
    * do NOT modify schema
    * DO show up in Admin UI, if used

##### Deconstructing - in use dynamic fields:

- No dynamic fields are used
    - * is a `copyField` instruction
- Can remove them all
    - `xml ed -L -d "//dynamicField" managed-schema`

> The number of lines drops from 481 to 409.

##### Deconstructing - field types:

* How many types out of 71 do we use? (9 field type definitions)
    - xml sel -t -m "//field|//dynamicField" -v "@type" -n conf/managed-schema | sort -u
    - long, string, strings, tdate, text_genereal
* But also some in solrconfig.xml
    - booleans, string, strings, tdates, tdoubles, text_general, tlongs

* Delete the rest (by hand)

> The number of lines from managed-schema dropes from 409 to 34


##### Deconstructing - support files:

* Inside lang directory (38 files)
    
- `find lang -name 'stopwords_*.txt' | wc -l`
    - stopwords_*.txt: 30 files
    - contractions_*.txt: 4 files
- `find lang -type f | egrep -v 'stopwords_|contractions_'`
    - hyphenations\_ga.txt
    - stemdict\_nl.txt
    - stoptags\_ja.txt
    - userdict\_ja.txt

* Support files, still in use?

- check for usage:
```
grep -o 'stopwords_*.txt' managed-schema solrconfig.xml
grep -o 'contraction_.*.txt' ...
```

> NO MATCHES (we no longer have related types).
    
- Delete the whole _lang_ directory.

* What about files just inside config directory?
    - Don't need `currency.xml`, `protwords.txt`


> down to 2 .txt files and 2 .xml files.


##### The mystery of \_root\_

* No explanations
* For nested documents (documentation):
    - schema must include an indexed/non-sorted field `_root_`...
* We are not using nested documents (no examples for solr)

* Remove the `_root_` from the managed-schema

> managed-schema downed from 34 to 33


##### Deconstruction - `text_general` type

* `text_general` support files:
    - `stopwords.txt`: just comments

* If you have no file, it will use a default list (default stopwords: list is English).
* Empty file, disabled stopwords (you get nothing).

* `text_general` simplified definition:
```
<fieldType name="text_general" class="solr.TextField" positionlncrementGap="100" multiValued="true">
    <analyzer>
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
    </analyzer>
</fieldType>
```

- Removed stopwords and synonyms:
> no more .txt files, and 26 lines in managed-schema


##### Deconstructing - solrconfig.xml

* solrconfig.xml is more complex than schema:
    - Heterogeneous Sections
    - Nested definitions
    - Alternative implementations (eg. highlighter)
    - Also remember:
        - configoverlay.json - overrides solrconfig.xml
        - params.json - additional configuration parameters

* solrconfig.xml feature counts:
    - 11 requestHandler
    - 8 lib
    - 5 searchComponent
    - 3 queryResponseWriter
    - 2 initParams
    - 1 updateRequestProcessorChain, updateHandler, requestDispatcher, query, luceneMatchVersion, jmx, indexConfig, directoryFactory, dataDir, codecFactory


##### add-unknown-fields-to-the-schema

* Famous "schemaless" mode (this is the main reason for it)
* Generic, but fully configurable
* Far from perfect (for development)
* Has normalization side-effects

> Cannot remove it in ou example

##### solrconfig.xml - highlighter

* Fragmenters
* encoders
* fragListBuilders
* fragmentBuilders
* boundaryScaners
* ...

> Can remove the **WHOLE** definition


> solrconfig.xml lines down from 278 to 226


##### Other searchComponents:

* Not on the default stack:
    - spellcheck
    - term
    - termVector
    - elevator

* Have dedicated **requestHandlers**
* Inception (example within example).

> Can be deleted (also delete elevate.xml)


> Down to one .xml file, and 163 lines in solrconfig.xml


##### More stuff:

* query section - can be removed since it has to be tuned
* updateHandler - revert to basic commits (lose smart-commit)
* jmx
* enableRemoteStreaming - take that out (don't put it into production)

* Keep velocity, browse, search support


(mailing list at)[www.solr-start.com]



