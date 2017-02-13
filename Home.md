[Download Latest Version of Hector](https://github.com/sastafford/hector/releases/latest)

# Introduction

Hector is a command line utility that imports delimited text files into MarkLogic.  Hector is similar to [MarkLogic Content Pump](http://docs.marklogic.com/guide/mlcp/intro) in terms of performance and the parameters used but differs in several ways.  

 * Transform content before ingestion     
 * Greater customization of URI generation

Both of these features are implemented via subclasses or implementing predefined interfaces.  
 
# Command Line Summary

Parameter | Optional | Description
----------|----------|-------------
input_file_path | No | Path for the file to be ingested
delimited_root_name | Yes | Root element for XML documents, only if document_type=xml, default value is "record"
document_type | Yes | Default is XML, can be "xml" or "json"
output_collections | Yes | Add this collection to each row in delimited file
uri_id | Yes | Specify one column as the URI, make sure that the values of this column are unique
uri_transform | Yes | Full package name for Java class that implements UriGenerator interface
output_transform | Yes | Full package name for Java class that implements ColumnMapSerializer interface
thread_size | Yes | Default value is 4
chunk_size | Yes | Default value is 5

# Hector Configuration

Hector requires a [job.properties](https://github.com/sastafford/hector/blob/master/job.properties) file to be defined and accessible in your classpath.

    marklogic.host=localhost
    marklogic.port=8200
    marklogic.username=admin
    marklogic.password=admin
    marklogic.name=hector-content

The property _marklogic.name_ corresponds to the name of your MarkLogic database.  

# Examples

The following examples use the Most Popular Baby Names from NYC CSV file contained under the test resources directory.  You can substitute this file and the parameters for the CSV file of your choosing.  Make sure that for each of your examples, you have a job.properties in your classpath or in your execution directory.    

## Example 1 - Import CSV File

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby  output_collections=baby
   
## Example 2 - Import CSV File as JSON

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv output_collections=baby document_type=json

## Example 3 - Import CSV with Content Transform

To apply a Java based transform to delimited files you must first be able to compile and build Hector.  Please refer to the [README](https://github.com/sastafford/hector/blob/master/README.md) for guidance. 

For XML documents, the first step is to add a Java class in the src/main folder under a package of your choosing.  The class will extend the [XmlStringColumnMapSerializer](https://github.com/sastafford/hector/blob/master/src/main/java/com/marklogic/hector/XmlStringColumnMapSerializer.java).  and the class you will override will be the following method. 

     protected Map<String, Object> transformColumnMap(Map<String, Object> columnMap)

The columnMap map variable contains the header labels in the key value and the corresponding value for that particular row in the map value.  The [BabyNameColumnMapSerializer](https://github.com/sastafford/hector/blob/master/src/test/java/com/marklogic/hector/BabyNameColumnMapSerializer.java) is an example of a Java delimited row transform.  

Once you have added the transform, then rebuild the package using the following gradle command.  

    gradle distZip

Your new distribution will be found under the build/distributions directory.  You can then reference your transform via the _output_transform_ parameter.  

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby document_type=xml output_collections=baby output_transform=com.marklogic.hector.examples.babies.BabyNameColumnMapSerializer

## Example 4 - Import CSV with URI Generation

Using the _uri_id_ parameter you can specify a column as the unique URI for the document.  For our baby names file we will use the NM (represents the NAME) as the uri.  

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby document_type=xml output_collections=baby output_transform=com.marklogic.hector.examples.babies.BabyNameColumnMapSerializer uri_id=NM

In this case, the baby name is not unique, therefore, documents are overwritten.  Suppose you have the requirement to concatenate multiple column names as the URI.  Similar to the transform, Hector has the capability of implementing an Interface to facilitate uri generation.  The [BabyNameUriGenerator](https://github.com/sastafford/hector/blob/master/src/test/java/com/marklogic/hector/BabyNameUriGenerator.java) class implements the [URIGenerator](https://github.com/sastafford/marklogic-spring-batch/blob/master/infrastructure/src/main/java/com/marklogic/spring/batch/item/processor/support/UriGenerator.java) interface.  Include this with your modified Hector distribution and rebuild your distribution.  It can then be referenced via the command line. 

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby document_type=xml output_collections=baby uri_transform=com.marklogic.hector.examples.babies.BabyNameUriGenerator

# Troubleshooting

 * Server Message: XDMP-DOCSTARTTAGCHAR: xdmp:get-request-part-body("xml") 
   * Make sure that your delimited text file is saved with UTF-8 encoding.  
 * Exception in thread "ThreadPoolTaskExecutor-x" com.sun.jersey.api.client.ClientHandlerException: org.apache.http.NoHttpResponseException: _host:port_ failed to respond
   * Decrease your thread count or chunk size.  The server resources are getting overwhelmed.
