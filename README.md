# pdfid.py

Changes from the original version:

* Add support for pseudo files
* Make it a python package 

-----------------------------------------------

[This documentation is a copy of the description available in the official website](https://blog.didierstevens.com/programs/pdf-tools/)

This tool is not a PDF parser, but it will scan a file to look for certain PDF keywords, allowing you to identify PDF documents that contain (for example) JavaScript or execute an action when opened. PDFiD will also handle name obfuscation.

The idea is to use this tool first to triage PDF documents, and then analyze the suspicious ones with my pdf-parser.

An important design criterium for this program is simplicity. Parsing a PDF document completely requires a very complex program, and hence it is bound to contain many (security) bugs. To avoid the risk of getting exploited, I decided to keep this program very simple (it is even simpler than pdf-parser.py).

![Printscreen](/img/20090330-214223.png)

PDFiD will scan a PDF document for a given list of strings and count the occurrences (total and obfuscated) of each word.

Almost every PDF documents will contain the first 7 words (obj through startxref), and to a lesser extent stream and endstream. I’ve found a couple of PDF documents without xref or trailer, but these are rare (BTW, this is not an indication of a malicious PDF document).

```
* obj
* endobj
* stream
* endstream
* xref
* trailer
* startxref
```

* `/Page`: gives an indication of the number of pages in the PDF document. Most malicious PDF document have only one page.
* `/Encrypt`: indicates that the PDF document has DRM or needs a password to be read.
* `/ObjStm`: counts the number of object streams. An object stream is a stream object that can contain other objects, and can therefor be used to obfuscate objects (by using different filters).


The two following indicate that the PDF document contains JavaScript. Almost all malicious PDF documents that I’ve found in the wild contain JavaScript (to exploit a JavaScript vulnerability and/or to execute a heap spray). Of course, you can also find JavaScript in PDF documents without malicious intend.

* `/JS`
* `/JavaScript`


The two following indicate an automatic action to be performed when the page/document is viewed. All malicious PDF documents with JavaScript I’ve seen in the wild had an automatic action to launch the JavaScript without user interaction.

* `/AA`
* `/OpenAction`

The combination of automatic action  and JavaScript makes a PDF document very suspicious.


* `/JBIG2Decode`: indicates if the PDF document uses JBIG2 compression. This is not necessarily and indication of a malicious PDF document, but requires further investigation.
* `/RichMedia`: is for embedded Flash.
* `/Launch`: counts launch actions.
* `/XFA`: is for XML Forms Architecture.


A number that appears between parentheses after the counter represents the number of obfuscated occurrences. For example, `/JBIG2Decode 1(1)` tells you that the PDF document contains the name `/JBIG2Decode` and that it was obfuscated (using hexcodes, e.g. `/JBIG#32Decode`).

BTW, all the counters can be skewed if the PDF document is saved with incremental updates.

Because `PDFiD` is just a string scanner (supporting name obfuscation), it will also generate false positives. For example, a simple text file starting with `%PDF-1.1` and containing words from the list will also be identified as a PDF document.




