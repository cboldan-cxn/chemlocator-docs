ChemLocator makes use of several ChemAxon tools for extracting the chemical molecules from various file formats. The following file formats are supported:

- ChemAxon MRV
- JChem, ISIS or ChemDraw for Excel workbook
- [Document to Structure](https://docs.chemaxon.com/Document_to_Structure_User_Guide.html) processes PDF, HTML, XML, text files and office file formats: DOC, DOCX, PPT, PPTX, XLS, XLSX, ODT. It recognizes and converts the chemical names (IUPAC, CAS, common and drug names), SMILES and InChI found in the document into chemical structures.
- [Document to Structure](https://docs.chemaxon.com/Document_to_Structure_User_Guide.html) converts the chemical structures from OLE objects – created by various chemical sketchers such as Marvin, ChemDraw, ISIS/DRAW, SYMYX DRAW, and Accelrys Draw – embedded in office documents.
- MDL SD File (this group of files may include filetypes having different extensions, for example, .rgf, which means SDF format containing R-group definitions or .csrgf, which is the compressed format of .rgf.)
- MDL RD File
- List of SMILES strings
- List of ChemAxon extended smiles strings
- List of IUPAC names
- ISISDraw sketch file - SKC
- ChemDraw sketch file - CDX, CDXML


For structures represented as images in PDF or Office documents, [Document to Structure](https://docs.chemaxon.com/Document_to_Structure_User_Guide.html) can make use of several Image to Structure tools (also called Optical Structure Recognition or Chemical OCR).

Currently, the supported Image to Structure tools are:

- Keymodule [CLiDE](http://www.keymodule.co.uk/products/clide/clide-batch.html)
- NIH [OSRA](https://cactus.nci.nih.gov/osra/)
