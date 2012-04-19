# X.commerce Open Commerce Language

Copyright (c) 2011 X.commerce

## License

By downloading any files from this repository, you are agreeing to the terms and conditions of the license that govern the X.commerce Contrarcts.  The license is available at [https://www.x.com/developers/xcommerce/products/developer-package/license](https://www.x.com/developers/xcommerce/products/developer-package/license).

## What's is XOCL?

The XOCL domain specific language (DSL) is a language developed at X.commerce that is used to describe business processes.  It is also known as XOCL Choreography because it describes how business processes are choreographed with business messages.  XOCL processes are defined in ".xocl" files and describe the business roles that a participant in the process would need to perform to fulfill those roles.  XOCL processes are realized through the implementation of XOCL workflows, which are sequences of transactions where messages are exchanged between the various business roles. 


## How does XOCL Choreography work?

Although XOCL Choreography is a high level DSL, it can be distilled into the language of the X.commerce Fabric, the Apache Avro Interface Description Language, written in ".avdl" files.  We have provided an avdl generator tool that will compile ".xocl" files into ".avdl" files for your consumption over the X.commerce Fabric.

Included in this repository, you will fine that all ".avdl" files has been pre-generated for you based on its ".xocl" counterpart.  The ".xocl" files can be found under the individual project's _src/xocl_ folder.  The ".avdl" files can be found under the individual project's _src-gen/avro_ folder.

For a full example of the XOCL Choreography syntax, please refer to the "Example" project located in this repository.

If you wish to compile your own ".xocl" into ".avdl" files, please refer to the Compilation section below.

## The Approach

Definition and evolution of the X.commerce Open Commerce Language, will be done in a community setting with X.commerce providing guiding principles and sponsorship.  The approach:

* The X.commerce Open Commerce Language will be maintained in this GitHub repository
* An officially published version of the X.commerce Open Commerce Language will be available in sandbox and production environments so capabilities compliant with this version can be deployed to those environments
* Anyone in the community (including partners, developers and eBay, Inc. employees) can fork this repository, work on changes and make a pull request to propose merging them back into the official version
* Initially X.commerce will have a core team of committers
* Over time, X.commerce will support domain-specific committers and expand the pool of committers to others in the community based on their contributions

The versioning scheme for contracts, including compatibility and deprecation rules, will be published soon.

## What Now?

* Get the [X.commerce Developer Package](https://www.x.com/fabric-download)
* Try the X.commerce Innovate "Code to Capability" [tutorial](https://github.com/xcommerce/innovate-developer-demo)
* Fork the repo and make the ecosystem better!

## Details

### Compilation (XOCL -> AVDL -> AVPR)

To compile XOCL into AVDL, you will need to build the xocl-maven-plugin from the [X.commerce XOCL Tools](https://github.com/xcommerce/XOCL-Tools) repository.  That repository will generate for you the xocl-maven-plugin and grammar jars that are needed to compile the ".xocl" files in this repository.  We hope to have a publicly hosted maven repository where these tools artifacts will be made available soon.

Once the ".avdl" files are generated, the Eclipse plug-in in the [X.commerce Developer Package](https://www.x.com/fabric-download) lets you create capabilities directly from ".avdl" files.  However, if you're using a dynamic language with Avro - there are bindings for Python, Ruby, PHP - you'll want to compile an AVDL to an AVPR file that can be parsed by the AvroProtocol (or similarly-named) class in your language of choice.  To compile, you will need the the Avro tools jar built from the [Java Avro package](http://www.apache.org/dyn/closer.cgi/avro/).  Then you can compile like so:

    java -jar ~/packages/avro-src-1.6.3/lang/java/tools/target/avro-tools-1.6.3.jar idl <idl file>

### Language-specific Tweaks

We've made a couple of tweaks to both the PHP and Ruby Avro implementations to make them easier to use for our purposes.  Specifically, we use the format typically used for data file storage for encoding messages, which required us to add a way to "force" writing complete schemas into messages.  The PHP versions are in this repository, and a modified Ruby gem is available:
    gem 'avro', :git => "git://github.com/xcommerce/avro-ruby.git")
We plan to get appropriate patches into the official open source releases.

Here's an example of this in action for both PHP and Ruby, referencing [Inventory.avdl](https://github.com/xcommerce/innovate-developer-demo/blob/master/Inventory.avdl) from the Innovate developer tutorial.

PHP:

    <?php

    include_once 'avro.php';

    $schema = file_get_contents($argv[1]);    // read an avpr; use Inventory.avdl

    $p = AvroProtocol::parse($schema);        // parse it

    $schemata = $p->schemata;                 // get all the schemas

    // combine the schemas we need in a union schema
    // the "true" parameter says "include all the schemas in the encoded message"

    $s = new AvroUnionSchema(array("Item", "Items"),
    			     "com.x.product.inventory", $schemata, true);

    $datum_writer = new AvroIODatumWriter($s);
    $strio = new AvroStringIO();
    $dw = new AvroDataIOWriter($strio, $datum_writer, $s);

    $i = array("sku" => "123", "title" => "title1", "currentPrice" => "price", "url" => "url", "dealOfTheDay" => "0");
    $is = array("items" => array($i));
    $dw->append($is);
    $dw->close();


Ruby:

    #!/usr/bin/env ruby

    require 'rubygems'
    require 'avro'

    protocol_text = File.open(ARGV[0], "rb").read    # read an avpr; use Inventory.avdl

    protocol = Avro::Protocol.parse(protocol_text)   # parse it

    schemas = {}
    protocol.types.map {|t| schemas[t.name] = t}     # get all the schemas

    # combine the schemas we need in a union schema
    # the "true" parameter says "include all the schemas in the encoded message"

    schema = Avro::Schema::UnionSchema.new(["Item", "Items"], "com.x.product.inventory", schemas, true)

    buffer = StringIO.new
    writer = Avro::IO::DatumWriter.new(schema)
    dw = Avro::DataFile::Writer.new(buffer, writer, schema)
    item = {"sku" => "123", "title" => "title", "currentPrice" => "price", "url" => "url", "dealOfTheDay" => "0"}
    items = {"items" => [item]}
    dw << items
    dw.close

