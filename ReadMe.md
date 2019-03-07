

Single document
---------------
	1/ Display in the screen
The server will send the context, and we have to prepare the fileOutput.
So, the variable has two children :
	"src" : this is the current file, is any (may be null if there are no existing file)
	"setdoc" : this is the new value, if user want to change the content

So the result is
var result =  {
  "oneFileDocumentInput" : 
		{ 'src':$data.context.oneFile_ref,'setdoc':null},
  
Note1 : you must set the setdoc. It must exist and be null to pass the contract. If the user does not select any new content, attribut will exist and is null.
Note2 : RestAPIContext prepare for you this format, and then you have nothing to write in the javascript

	2/ Widget
You have to place a Link Widget to access the current version, and a FileUpload widget to capture the new version
LinkWidget : 
	name : {{formInput.oneFileDocumentInput.src.fileName}}
	Url :  "/bonita/portal/" + formInput.oneFileDocumentInput.src.url

FileUpload widget:
	value: formInput.oneFileDocumentInput.setdoc

Note: the widget FileUploadPlus will do this two construction, plus add a button "clear" to let the user clear the content.
With the default widget, you don't have any way to implemente the Clear function.
FileUploadPlus widget :
	value : formInput.oneFileDocumentInput

	3/ formOutput
Nothing to do : just return $data.formInput

	4/ Contract
The contract must be
	oneFileDocumentInput COMPLEX
		setdoc	FILE
Note : you must not set the "src" because if there are not yet any 

	5/ operation
The groovy script to update the SingleFileDocument is:

import org.bonitasoft.engine.bpm.document.DocumentValue


if (oneFileDocumentInput.get("setdoc") != null) {
  // we have a new content	
  return new DocumentValue(oneFileDocumentInput.get("setdoc").getContent(), oneFileDocumentInput.get("setdoc").getContentType(), oneFileDocumentInput.get("setdoc").getFileName());
} else if (oneFileDocumentInput.get("src") != null) {
  // user keep the current value
  return new DocumentValue(oneFileDocumentInput.get("src").get("id"));
} else {
  return null
}

 

List of documents
-----------------
	You want to display a list of document, and the user can add new document, change the content, remove a document.
	
		1/ Display in the screen
Same principle, the server will send the context, and we have to prepare the fileOutput.
So, the list of item has two children :
	"src" : this is the current file, is any (may be null if there are no existing file)
	"setdoc" : this is the new value, if user want to change the content

So the result is
var result =  {
var listDocs=[];
for (var oneDoc in $data.context.oneListFile_ref)
{
    var oneLine ={ 'src': $data.context.oneListFile_ref[ oneDoc], 'setdoc':null };
    listDocs.push( oneLine );
}
 var result = { "oneListFileDocumentInput" : listDocs }
 
 
Note1 : you must set the setdoc. It must exist and be null to pass the contract. If the user does not select any new content, attribut will exist and is null.
Note2 : RestAPIContext prepare for you this format, and then you have nothing to write in the javascript

	2/ Widget
	You have to place a container widget to display the list. 
Container widget:
	collection : formInput.oneListFileDocumentInput
Link Widget
	name : {{$item.src.fileName}}
	Url :  "/bonita/portal/" + $item.src.url

FileUpload widget:
	value: $item.setdoc

Note: the widget FileUploadPlus will do this two construction, plus add a button "clear" to let the user clear the content.
With the default widget, you don't have any way to implements the Clear function.
FileUploadPlus widget :
	value : $item

	3/ formOutput
Nothing to do : just return $data.formInput

	4/ Contract
The contract must be
	oneFileDocumentInput COMPLEX MULTIPLE
		setdoc	FILE
Note : you must not set the "src" because if there are not yet any 

	5/ operation
The server will send a List of Document like.

Exemple, server send doc1,doc2

The list is then
[ 
   { 
      "src":{ 
         "id":408,
         "processInstanceId":4004,
         "name":"oneListFile",
         "author":4,
         "creationDate":1478799273137,
         "fileName":"Doc 1.docx",
         "contentMimeType":"application/vnd.openxmlformats-officedocument.wordprocessingml.document",
         "contentStorageId":"408",
         "url":"documentDownload?fileName=Doc 1.docx&contentStorageId=408",
         "description":null,
         "version":"1",
         "index":0,
         "contentFileName":"Doc 1.docx"
      },
      "setdoc":{ 
         "tempPath":"tmp_86221273153056760.docx",
         "filename":"Doc 4.docx",
         "contentType":"application/vnd.openxmlformats-officedocument.wordprocessingml.document"
      }
   },
   { 
      "src":{ 
         "id":410,
         "processInstanceId":4004,
         "name":"oneListFile",
         "author":4,
         "creationDate":1478799273140,
         "fileName":"Doc 3.docx",
         "contentMimeType":"application/vnd.openxmlformats-officedocument.wordprocessingml.document",
         "contentStorageId":"410",
         "url":"documentDownload?fileName=Doc 3.docx&contentStorageId=410",
         "description":null,
         "version":"1",
         "index":2,
         "contentFileName":"Doc 3.docx"
      }
   }
]
 
For each item, we have then
  "src": if present, this is the current item. Then, src.id is the current documentid
  "setdoc" : this is the new document uploaded.


 
The operation to update the list of document is 


import java.util.logging.Logger;

import org.bonitasoft.engine.bpm.document.DocumentValue

List<DocumentValue> listResult= new ArrayList<DocumentValue>();
 
Logger logger = Logger.getLogger("org.bonitasoft");

for (int i=0; i<oneListFileDocumentInput.size();i++)
{
	if (oneListFileDocumentInput.get( i )== null) {
		 continue; // should never arrive
	}
	if (oneListFileDocumentInput.get( i ).get("setdoc") !=null)
	{
		def newFileDocumentInput = oneListFileDocumentInput.get( i ).get("setdoc")
		// we have a new content
		logger.info("Line ["+i+"] getFileName["+newFileDocumentInput.getFileName()+"] class ["+newFileDocumentInput.getClass().getName()+"]")
		listResult.add( new DocumentValue( newFileDocumentInput.getContent(), newFileDocumentInput.getContentType(), newFileDocumentInput.getFileName()));
		
	} else if (oneListFileDocumentInput.get( i ).get("src") !=null)
	{
		// we want to keep the current document in this position
		logger.info("Line ["+i+"] Keep Document ["+oneListFileDocumentInput.get( i ).get("src").get("id")+"]");
		listResult.add( new DocumentValue( oneListFileDocumentInput.get( i ).get("src").get("id") ) );
	}
	else
	{
		// user give a new empty line : no new value, no old value, just ignore it
	}
}

return listResult;
