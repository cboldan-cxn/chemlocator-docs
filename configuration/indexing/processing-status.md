# Processing status

The Processing status page can be reached by opening the Administration web 
pages. Then under the Indexing menu, click the Processing status item.

The page describes the current status of the currently running application tasks.

## Indexing

This section shows information about the indexing process.

### New items

It shows the number of the documents which are waiting to be indexed but the 
processing has not started yet.

### Processing items

It shows the number of the documents which are under processing. During 
indexing, the number of these documents depends on the Application settings 
(asynchronous processing settings).

### Error items

It shows the number of the documents which failed during the processing. The 
error can be found in the indexing log. More information can be found here: 
Monitoring errors, events.

### Complete items

Shows the successfully processed item count. It will show all the documents 
which have been processed since the last index reset/database reset.

**TODO: image**

## Query

The section shows audit information about the general usage of the ChemLocator 
app.

  - **in the last minute** - shows the number of queries the users issued in the 
    past 60 seconds
  - **in the last hour** - shows the number of queries the users issued in the 
    past 60 minutes
  - **in the last 24 hours** - shows the number of queries the users issued in 
    the past 24 hours
  - **in the last 7 days** - shows the number of queries the users issued in the 
    past 7 days

**TODO: image**

## Annotation

The section shows audit information about the document annotation process of the 
ChemLocator app. Document annotation process is a multi-step processing. 
Document conversion (PDF to HTML, in some cases XML to HTML) and document 
annotation (highlighting chemical structures) belong to it.

### Total documents

Shows the number of all documents within the ChemLocator database available for 
annotation.

### Annotated documents

Shows the number of documents on which the annotation process already finished 
successfully.

### Remaining documents

Shows the number of documents which are still in the annotation queue and 
waiting for processing.

### View current items

Shows the documents which are under processing at the moment

### View errors

The number of documents which had errors during the annotation processing.