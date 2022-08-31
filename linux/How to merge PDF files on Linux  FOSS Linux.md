PDF files are essential for both personal and professional documents. There are several command-line and GUI Linux tools you can use to combine multiple inter-related PDF files into a single PDF file.

The article is a step-by-step guide on merging multiple PDF documents or pages into one PDF without breaking the PDF content. The demonstration will use open-source, free, command-line, and GUI applications.

## Merge Multiple PDF Files in Linux Command Line

Combining PDF files from the command-line is essential to system administrators who work on a server without a GUI. You can use several command-line tools such as PDFtk, Ghostscript, Convert ImageMagick Tool, and pdfunite.

#### PDFtk

[PDFtk](https://www.pdflabs.com/) is a free command-line tool to merge several pdf files. PDFtk is available in three variants:

-   PDFtk Free: a free graphical app
-   PDFtk Server: a free command-line tool
-   PDFtk Pro: paid version with both CLI and GUI app

PDFtk provides the following functionality:

-   You can merge PDF files or collate PDF page scans.
-   You can split multiple PDF pages into a new document.
-   You can edit PDF file metadata.
-   You can manipulate and rotate PDF pages.
-   It allows you to add a foreground stamp or a background watermark.
-   You can fill PDF Forms with X/FDF data or Flatten Forms.
-   You can also attach files to PDF pages and unpack PDF attachments.

##### Install PDFtk On Linux

###### Ubuntu & Debian

You can install PDFtk on Debian and Ubuntu-based Linux distros with apt using the following command.

```
$ sudo apt install pdftk-java
```

###### Fedora, CentOS, and Red Hat

The first step is to install the libgcj dependency.

```
$ sudo yum install libgcj
```

Download the Binary RPM file (available for both 64-bit and 32-bit architecture) with curl or wget.

```
wget https://www.pdflabs.com/tools/pdftk-the-pdf-toolkit/pdftk-2.02-1.el6.x86_64.rpm
```

Install the RPM file.

```
$ sudo rpm -i pdftk-2.02-1.*.rpm
```

###### Snap

```
$ sudo snap install pdftk
```

##### Combine PDFs with PDFtk

To combine several PDFs, you need to provide the names of the files and the output name of the single combined PDF. The command will create a new PDF file named “mypdf3.pdf” that will have the merged contents of both “mypdf1.pdf” and “mypdf2.pdf” files.

```
$ pdftk mypdf1.pdf mypdf2 cat output mypdf3.pdf
```

![PDFtk](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDFtk

The command above is suitable for scenarios where you only have a few PDF files to combine. However, if you have a large number of PDF pages, you can use an asterisk (\*) wildcards to indicate all the PDFs in your current working directory. For example, use \*.pdf to show all the files with the .pdf extension. It will save you the effort of writing all the file names separately.

```
$ pdftk *.pdf cat output ALL_COMBINED.pdf
```

###### Encrypt a PDF file PDFtk

You can use PDFtk to encrypt a PDF file with the owner\_pw option.

```
$ pdftk unsecured-1.pdf output secured-1.pdf owner_pw XYZ [Encrypt a PDF file]
```

###### Decrypt a PDF file with PDFtk

You can then decrypt the PDF file (secured-1.pdf) with the input\_pw option.

```
$ pdftk secured-1.pdf input_pw xyz output unsecured.pdf [Decrypt a PDF file]
```

Learn more tricks and tips like removing and deleting pages from PDF from [PDFtk official manual pages.](https://www.pdflabs.com/docs/pdftk-man-page/)

#### Convert ImageMagick Tool

[ImageMagick](https://imagemagick.org/) is primarily an image optimization tool. However, it also includes a conversion tool to merge multiple PDFs.

##### Install ImageMagick

###### Debian & Ubuntu-based distros

```
$ sudo apt install imagemagick
```

###### Fedora

```
$ sudo dnf install ImageMagick
```

##### CentOS / Red Hat

```
$ sudo yum install ImageMagick
```

##### Merge PDFs with ImageMagick

![Convert Image Magick](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

Convert Image Magick

To merge multiple PDFs, you need to provide the file names of the original PDFs to be merged, then the filename for the final merged PDF file. The command will create a new PDF file named “final\_pdf.pdf” that will have the merged contents of “pdf1.pdf”, “pdf3.pdf,” and “pdf2.pdf” files.

```
convert pdf1.pdf pdf3.pdf pdf2.pdf final_pdf.pdf
```

##### Merge specific pages from PDFs

You can merge specific pages by indicating the pages starting from 0. For example, you can combine pages 1-2 from one PDF with a second pdf file.

```
convert pdf1.pdf[0-3] pdf2.pdf[5-7] final_pdf.pdf
```

#### Ghostscript

[Ghostscript](https://www.ghostscript.com/) is a versatile CLI app for manipulating PDF, PostScript, and XPS files.

##### Install Ghostscript

###### Debian & Ubuntu-based distros

```
$ sudo apt-get install ghostscript
```

##### Fedora

```
$ sudo dnf install ghostscript
```

##### CentOS & Red Hat

```
$ sudo yum install ghostscript
```

##### Combine PDFs with the gs command

To merge multiple PDFs, run the following gs command:

```
# gs -dNOPAUSE -sDEVICE=pdfwrite -sOUTPUTFILE=merged_file.pdf -dBATCH pdf_1.pdf pdf_2.pdf
```

Notes:

-   use -dNOPAUSE option to disable continuation prompts at the end of each PDF page.
-   Use the -sDEVICE attribute to specify the output device or function.
-   Use the -sOUTPUTFILE to specify the merged PDF file.
-   Use the -dBATCH to specify the PDF files to combine in the order you want them to appear.
-   The command above will output merged\_file.pdf as a combination of pdf\_1.pdf and pdf\_2.pdf files.

#### pdfunite

pdfunite by [Poppler](https://poppler.freedesktop.org/) is yet another command-line utility to merge multiple PDFs. It is natively available on Ubuntu-based, Arch, Mint, and Manjaro distros. The popper-utils package provides several commands for modifying PDF files like the pdfseparate and pdfunite commands.

##### Install poppler-utils package

To use pdfunite, you need to install the “poppler” utility with the following command:

###### Debian / Ubuntu-based distros

```
$ sudo apt install poppler-utils
```

##### Fedora, CentOS

```
$ sudo dnf install poppler-utils
```

##### Extract pages into multiple PDFs with pdfseparate command

You can use pdfseparate to extract pages into multiple PDFs that you can later merge with pdfunite.  
Use the following command to extract all the pages into individual PDF pages.

```
# pdfseparate final_pdf.pdf final_pdf-page_%d.pdf
```

You can also export a range of pages. For example, use the following command to extract pages 25,26,27,28,29,30 and 31 of a PDF document.

```
pdfseparate -f 25 -l 31 FOSSBook.pdf FOSSBook-page_%d.pdf
```

##### Combine PDFs with pdfunite command

![pdfunite](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

Merge PDFs with pdfunite

The pdfunite command uses the same format ImageMagick tool. The last filename (merged\_file.pdf) indicates the new output file. All the PDF files listed before it are the files you want to merge. After the command completes, the combined PDF file is named “merged\_file.pdf” will be an integrated version of all files indicated before it.

```
# pdfunite pdf_1.pdf pdf_2.pdf merged_file.pdf
```

### Merge Multiple PDF Files using GUI applications

There are several popular desktop apps to merge PDF files. Some apps include PDF Arranger, LibreOffice Draw, PDF Chain, PDFSam, PDF Shuffler, and PDFmod.

#### PDF Arranger

[PDF Arranger](https://github.com/pdfarranger/pdfarranger) includes the following features and functionality.

-   Merge multiple PDF documents
-   Reorder PDF pages
-   Export all or several pages from a PDF file
-   Duplicate PDF pages
-   Delete, rotate, and crop PDF pages
-   Edit PDF metadata
-   Zoom in and out

##### Install PDF Arranger

###### Flatpak

If you can install PDF Arranger using flatpak with the following command. Before you start, make sure you have Flatpak running in your system.

```
$ flatpak install flathub com.github.jeromerobert.pdfarranger
```

##### Combine PDFs with PDF Arranger

Once you install it successfully, open the app and click on the icon at the top left corner. It will open a dialog box to select all PDFs you want to combine.

![PDF Arranger](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDF Arranger: Import PDFs

Now you can see a list of all pages from the selected PDFs. You can then manipulate, rearrange, delete, export, and edit metadata of the pages before you combine them into one PDF document.

![PDF Arranger](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDF Arranger: merged PDF

#### PDF Chain

[PDF Chain](https://flathub.org/apps/details/net.sourceforge.pdfchain) is a GUI for the PDFtk command-line utility. It is open-source and is written in C++. Its graphical user interface gives you access to most of PDFtk’s commands.

Its features include:

-   Merge PDF files (maximum of 26 files).
-   Select several or contiguous pages.
-   Rotate PDF pages.
-   Split a PDF document into separate pages.
-   Add a background or watermark to a PDF file.
-   Add attachments to a PDF file.
-   Setting permissions for output PDF file.
-   Setting user or owner password.
-   Setting encryption and decryption.

PDF Chain also features tools that allow you to:

-   Extract attachments from a PDF
-   Extract PDF metadata
-   Dump data and data fields
-   Compress or uncompress a file
-   Flatten a PDF document
-   Fill in PDF forms
-   Drop XML Forms Architecture (XFA) data from PDF forms

##### Install PDF Chain

###### Fedora

```
flatpak install flathub net.sourceforge.pdfchain
```

Run PDF Chain.

```
flatpak run net.sourceforge.pdfchain
```

##### Merge PDFs with PDF Chain

![PDF Chain](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDF Chain: Combine PDFs

Click on the ‘+’ button at the bottom left corner, select your PDFs using Shift + Click or Ctrl + Click to select multiple pages. Finally, click ‘save as’ at the bottom right corner to save your merged pdf document.

#### PDF Shuffler

[PDF Shuffler](https://linux.die.net/man/1/pdfshuffler) is a GUI app to move and rearrange pages in a PDF document. It has limited functionality. However, you can use it to:

-   Extract pages from PDF documents
-   Add pages to a PDF file
-   Rearrange the pages in a PDF file

##### Install PDF Shuffler

###### Fedora

```
$ sudo dnf install pdfshuffler
```

##### Ubuntu /Debian

```
$ sudo apt install pdfshuffler
```

![PDF Shuffler](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDF Shuffler: Merge PDFs

To extract pages from a PDF file, open it by selecting: File>Add.

To extract pages 3 to 5, press Ctrl and click-select the pages. Then, right-click and select the Export option. Next, select a location to save, give it a name, then click save.

![PDF Shuffler](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDF Shuffler: Final Merged PDF document

To add a PDF file, open it, select: File > Add and find the PDF file you want to add. Click Open. To complete, click and drag the page you added to the desired location in the file. Note that you can only click and drag one page at a time.

#### PDFmod

[PDFmod](https://pkgs.org/download/pdfmod) is very much similar to the PDFShuffler application. They function pretty much the same way. Once you have imported PDFs into PDFmod, it will display all the pages in the document, ready for you to modify.

##### Install PDFmod

###### Fedora

```
$ sudo dnf install pdfmod
```

###### Ubuntu

```
$ sudo apt install pdfmod
```

![PDFmod](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDFmod

To rearrange pages, select them using Shift + Click or Ctrl + Click to select multiple pages, then drag ‘n’ drop them to their desired location in the PDF document.

To remove pages, select using Shift + Click or Ctrl + Click to select multiple pages, then press delete. When done, save your document as a new PDF file.

#### LibreOffice Draw

LibreOffice Writer does not allow you to combine several PDFs. However, you can achieve the same with [LibreOffice Draw](https://www.libreoffice.org/discover/draw/).

##### Install LibreOffice Draw

###### Fedora

```
$ sudo dnf install libreoffice-draw
```

##### Merge PDFs with LibreOffice Draw

You can merge PDFs with LibreOffice using the following simple workaround steps.

Step 1: Open your first PDF document in LibreOffice Draw, resize and drag the window to fill the left half of your display screen.

![libreofficedraw1](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

LibreOffice Draw: Open PDF 1st PDF

Step 2: Open your second PDF document in a new LibreOffice Draw window, then resize and drag the window to fill the right half of your screen.

![libreofficedraw2](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

LibreOffice Draw: Open 2nd PDF

Step3: You will note that each window will display two columns. The left column is the pages pane that shows all the pages of each PDF document. Drag the pages from the first PDF into the pages pane of the second PDF. You can then order the pages however you wish.

![libofficedraw4](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

LibreOffice Draw: Merged PDFs

Step 4: Once you are done rearranging the pages of the combined PDF, click File → Export As → Export Directly as PDF. It will generate a new PDF file in your current working directory.

#### PDFSam

[PDFSam](https://pdfsam.org/) is yet another tool to modify and edit PDF documents on Linux.

##### Install PDFSam

###### Ubuntu

Download the official PDFSam DEB package to a local directory using the [wget command](https://www.fosslinux.com/49543/wget-linux-command.htm).

```
# wget https://github.com/torakiki/pdfsam/releases/download/v4.2.8/pdfsam_4.2.8-1_amd64.deb
```

Install the PDFSam DEB package using the apt install command.

```
$ sudo apt install ./pdfsam_4.2.8-1_amd64.deb
```

###### Debian

After downloading the latest release of PDFSam to your local directory, use the dpkg command the install the DEB package.

```
$ sudo dpkg -i pdfsam_4.2.8-1_amd64.deb
```

###### Fedora

Before you install PDFSam on Fedora 34 or newer, make sure you have Java installed for it to run.  
Download the latest release of PDFSam using the wget command.

```
# wget https://github.com/torakiki/pdfsam/releases/download/v4.2.8/pdfsam-4.2.8-linux.tar.gz
```

Extract the PDFSam package to your local directory with tar.

```
# tar xvf pdfsam-4.2.8-linux.tar.gz
```

Run PDFSam on Fedora with the following commands.  
Change your current working directory to pdfsam-4.2.8-linux.

```
# cd ~/pdfsam-4.2.8-linux
```

Run the PDFSam app.

```
# java -jar pdfsam-basic-4.2.8.jar
```

Merge PDFs with PDFSam

![PDFSam](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDFSam: Open ‘merge’ button

Step 1: Open the PDFSam app and click on the “Merge” button to open the merge menu.

Step 2: Inside the merge menu, find the PDFs you wish to merge using Linux file manager and Drag and drop the PDF files.

Step 3: After all PDF files are added to the PDFSam merge menu, you can change the merge settings.

![PDFSam](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDFSam: Click run to merge PDFs

Step 4: Click on the “Run” button at the bottom of the PDFSam page to create a new PDF from the files you’ve added to the merge menu. The merging process will create a new PDF file(PDFsam\_merge.pdf) when the merging process is complete.

![PDFSam](https://www.fosslinux.com/wp-content/plugins/wp-fastest-cache-premium/pro/images/blank.gif)

PDFSam : PDFSam\_merge.pdf file

## Wrapping up

You can quickly merge two or more PDF files in Linux via the command line or GUI apps. Apart from merging PDFs, some apps such as PDF Arranger and PDFtk provide additional functionality like editing metadata, adding a foreground stamp or a background watermark, and encrypting or decrypting your PDF documents.

You can also use LibreOffice Draw to rearrange and merge pages into a second PDF document. Based on your experience, these tools and methods are convenient ways to merge PDF files.