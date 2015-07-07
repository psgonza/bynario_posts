layout: post
title: "TIP: Script para convertir Doc a RFT"
date: 2011-01-19T15:41:00+01:00
category: posts
tag: scripts
slug: 2011-01-19-tip-script-para-convertir-doc-rft

He tenido que convertir todos los documentos doc que tenia en una carpeta a formato RTF (para que calibre me dejara pasarlos a distintos formatos).  

Como eran unos pocos, en lugar de hacerlo a mano, he usado un script en VBS para ahorrarme un rato.  

```
On Error Resume Next
Dim fso, folder, files, NewsFile,sFolder

Set fso = CreateObject("Scripting.FileSystemObject")
sFolder = Wscript.Arguments.Item(0)
If sFolder = "" Then
Wscript.Echo "No Folder parameter was passed"
Wscript.Quit
End If
Set NewFile = fso.CreateTextFile(sFolder&"\FileList.txt", True)
Set folder = fso.GetFolder(sFolder)
Set files = folder.Files

For each folderIdx In files
'NewFile.WriteLine(folderIdx.Name)
Doc2RTF folderIdx.Name 
Next
NewFile.Close


Sub Doc2RTF( myFile )
' This subroutine opens a Word document, then saves it as RTF, and closes Word.
' If the RTF file exists, it is overwritten.
' If Word was already active, the subroutine will leave the other document(s)
' alone and close only its "own" document.
'
' Written by Rob van der Woude
' http://www.robvanderwoude.com

' Standard housekeeping
Dim objDoc, objFile, objFSO, objWord, strFile, strRTF

Const wdFormatDocument = 0
Const wdFormatDocument97 = 0
Const wdFormatDocumentDefault = 16
Const wdFormatDOSText = 4
Const wdFormatDOSTextLineBreaks = 5
Const wdFormatEncodedText = 7
Const wdFormatFilteredHTML = 10
Const wdFormatFlatXML = 19
Const wdFormatFlatXMLMacroEnabled = 20
Const wdFormatFlatXMLTemplate = 21
Const wdFormatFlatXMLTemplateMacroEnabled = 22
Const wdFormatHTML = 8
Const wdFormatPDF = 17
Const wdFormatRTF = 6
Const wdFormatTemplate = 1
Const wdFormatTemplate97 = 1
Const wdFormatText = 2
Const wdFormatTextLineBreaks = 3
Const wdFormatUnicodeText = 7
Const wdFormatWebArchive = 9
Const wdFormatXML = 11
Const wdFormatXMLDocument = 12
Const wdFormatXMLDocumentMacroEnabled = 13
Const wdFormatXMLTemplate = 14
Const wdFormatXMLTemplateMacroEnabled = 15
Const wdFormatXPS = 18

' Create a File System object
Set objFSO = CreateObject( "Scripting.FileSystemObject" )

' Create a Word object
Set objWord = CreateObject( "Word.Application" )

With objWord
' True: make Word visible; False: invisible
.Visible = True

' Check if the Word document exists
If objFSO.FileExists( myFile ) Then
Set objFile = objFSO.GetFile( myFile )
strFile = objFile.Path
Else
WScript.Echo "FILE OPEN ERROR: The file does not exist" & vbCrLf
' Close Word
.Quit
Exit Sub
End If

' Build the fully qualified HTML file name
strRTF = objFSO.BuildPath( objFile.ParentFolder, _
objFSO.GetBaseName( objFile ) & ".rtf" )

' Open the Word document
.Documents.Open strFile

' Make the opened file the active document
Set objDoc = .ActiveDocument

' Save as HTML
objDoc.SaveAs strRTF, wdFormatRTF

' Close the active document
objDoc.Close

' Close Word
.Quit
End With
End Sub

```

Colocad el script en el directorio donde esten los ficheros.doc, y desde la consola de DOS ejecutad:

```
doc2rtf.vbs .
```

Saludos  


