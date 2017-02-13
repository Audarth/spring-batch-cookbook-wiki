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

The following examples use the Most Popular Baby Names from NYC CSV file contained under the test resources directory.  You can substitute this file and the parameters for the CSV file of your choosing.  

## Example 1 - Import CSV File

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby document_type=xml output_collections=baby
   
## Example 2 - Import CSV with Content Transform

    hector.bat input_file_path=./src/test/resources/Most_Popular_Baby_Names_NYC.csv delimited_root_name=baby document_type=xml output_collections=baby output_transform=com.marklogic.hector.examples.babies.BabyNameColumnMapSerializer

A transform can be defined by subclassing the XmlStringColumnMapSerializer.  See the BabyNameColumnMapSerializer for an example.  

# Troubleshooting

 * Server Message: XDMP-DOCSTARTTAGCHAR: xdmp:get-request-part-body("xml") 
   * Make sure that your delimited text file is saved with UTF-8 encoding.  

