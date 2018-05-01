
# Tika

Apache Tika is a toolkit for extracting metadata and text from over a thousand different file types (such as PPT, XLS, and PDF). This has significant value for Marklogic as it allows you to create indexable and searchable xml representations of your binary files. 

When combined with spring batch you can easily create a workflow to process any files with Tika and ingest the resulting xml document in to Marklogic. 

The following explains how to use the example marklogic-spring-batch Tika Job Configuration:
 
# build.gradle

In the `tika` folder you will find a build.gradle file which defines the java repositories as well as the task configuration for `importAndExtract`. 

You will notice that `importAndExtract` has a number of arguments defined:

Parameter | Optional | Description
----------|----------|-------------
input_file_path | No | Path for the file to be ingested
output_collections | No | A comma delimited list of collections to be added to the resulting document
input_file_pattern | Yes | An optional property to restrict which files are ingested from the file path.
thread_size | No| The number of threads which will be used
chunk_size | No| The number of documents which will be written in each batch

# Configuration

Tika requires a job.properties file to be defined and accessible in your classpath.

    marklogic.host=localhost
    marklogic.port=8000
    marklogic.username=admin
    marklogic.password=admin

# Examples

The following examples use the Most Popular Baby Names from NYC CSV file contained under the test resources directory.  You can substitute this file and the parameters for the CSV file of your choosing.  Make sure that for each of your examples, you have a job.properties in your classpath or in your execution directory.    

## Example 1 - Lorem Ipsum

To run the task as currently defined:
```
 gradlew importAndExtract
```

After job executes, there will be 1 document that exists in the target database in the 'tika' collection. The contents will be in an HTML format with the metadata and content extracted out of a Microsoft Word document (docx) from `./src/test/resources/doc/LoremIpsum.docx`.
   
## Example 2 - Creating a new Task

To create a task you simply need to reference the same job_path and job_id to use the defined java configuration then put in the relevant arguments as defined in  [Configuration](#configuration)

For example, to process all pdf files located in `/tmp/` and put them in collections `tikaPDF` and `pdf` you could add the following task to your `build.gradle` file.
```
task examplePdfExtract(type: JavaExec) {
    main = 'com.marklogic.spring.batch.core.launch.support.CommandLineJobRunner'
    classpath = sourceSets.main.runtimeClasspath + files("../")
    args = ["--job_path", "com.marklogic.batch.tika.ImportAndExtractContentJobConfig",
            "--job_id", "importAndExtractContentJob",
            "--input_file_path", "/tmp/",
            "--input_file_pattern", ".*pdf",
            "--output_collections", "tikaPDF,pdf",
            "--chunk_size", "100",
            "--thread_count", "4",
            "--next"]
}
```
Then run it with:

```
gradlew examplePdfExtract
```

# Troubleshooting

 * Exception in thread "ThreadPoolTaskExecutor-x" com.sun.jersey.api.client.ClientHandlerException: org.apache.http.NoHttpResponseException: _host:port_ failed to respond
   * Decrease your thread count or chunk size.  The server resources are getting overwhelmed.