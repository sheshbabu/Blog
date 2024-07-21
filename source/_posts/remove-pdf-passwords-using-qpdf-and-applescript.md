---
title: Remove PDF password protection using qpdf and AppleScript
date: 2024-07-21 17:13:30
keywords: PDF, AppleScript
image: /images/2024-qpdf-and-applescript/2024-qpdf-and-applescript.png
tags:
  - AppleScript
---

For removing password protection from PDFs, I prefer using [qpdf](https://github.com/qpdf/qpdf) instead of installing bloated applications. One downside is that it needs to be invoked from Terminal, which is okay but not super convenient. I found a way to use macOS Automator to call this tool from Finder. 

![](/images/2024-qpdf-and-applescript/2024-qpdf-and-applescript-01.png)

Install qpdf from [HomeBrew](https://formulae.brew.sh/formula/qpdf)

```shell
$ brew install qpdf
```

Open the builtin macOS application called “Automator". Select “New Document” in the file picker dialog and choose “Quick Action” as the document type.

Set `Workflow receives current` dropdown to "PDF files" and `in` "Finder". Drag and drop `Run AppleScript` action into the left pane. 

Add the following code and save this Quick Action as "Remove Password":

```applescript
on run {input, parameters}
	set the_file_path to POSIX path of (input as text)
	
	set AppleScript's text item delimiters to "/"
	set file_name to last text item of the_file_path
	set AppleScript's text item delimiters to "."
	set file_name_without_extension to first text item of file_name
	set AppleScript's text item delimiters to ""
	
	set the_password to display dialog "Enter the password for the PDF file:" default answer "" with icon caution
	set the_password to text returned of the_password
	
	set the_output_path to "/Users/" & (do shell script "echo $USER") & "/Downloads/" & file_name_without_extension & "_decrypted.pdf"
	
	do shell script "/opt/homebrew/bin/qpdf --password=" & quoted form of the_password & " --decrypt " & quoted form of the_file_path & " " & quoted form of the_output_path
	
	display dialog "The decrypted PDF file is saved as: " & the_output_path
end run
```

This will show up when you right-click a PDF file in Finder under Quick Actions. Once decrypted, it saves the decrypted file to the user's `Downloads` folder. 