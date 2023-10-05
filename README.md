# downloader-thingy
Get and correctly rename Black Equine comic file downloads (this is an obviously stupid name to avoid keywords).


This isn’t a program, it’s instructions on how to download files you’ve bought from a particular comic retailer that is a “black equine”.  It is also the worst kind of messy just-works shambles, with no effort beyond that.  It’s worked for me but no guarantees beyond that!

Upside, this might be worth it because the images are actually 1800x2700 – so I think slightly better than downloading from the “big river” place.

This store doesn't encrypt downloads at the moment - so I assume this isn't circumventing technical protections for anyone who cares - they simply obfusticate really obnoxiously.  When you download, the file comes are a tar archive, and with randomized out-of-order filenames, so you need to get the mapping of filenames to page numbers out of the json file. 

Easy done with grep.

For windows, there’s stuff on github to do this – I won’t link because the keyword I want to avoid is in the title, but its by Metalnem and it shows up in google.  I haven’t tried it, I don’t have windows so don’t know if it still works, but that's where this started from, and it might be easier than this if you are able to.

Otherwise, you need something to untar, and something to sed and grep, and bash – so all on linux, but I’m not sure for windows – I’d assume any of the linux utilities things would work.

So - 

Go to your library page, open a book, open that book to read it.  You need to be in the web view reading pages.

Do inspect/view source in your brower (CTRL+U in firefox).

Do find (CTRL + F) and search for "manifest.jsonp".

Click the link that contains it (long randomised filename) to open the whole json file in the browser.  You see a bunch of plain text.

Look for the line starting "book_uuid": at the top of the page.  It will be something like - 

	"book_uuid": "123456789abcde123456789"

Copy the aplhanumeric code string in the parenthesis and paste it into this -

	https://digital.[equine].com/api/v6/book/

To make something like this - 

	https://digital.[equine].com/api/v6/book/123456789abcde123456789

Open that in your browser and a file called “book.tar” should start downloading.

!!!  CHANGE [equine] TO THE ACTUAL STORE URL OBVOUSLY  !!!


You should now have a tar file that has the jpgs in it (but not named jpg, but they should still open anyway with a sensible image viewer so you can check).


Next you need this as a shell script –


	#!/bin/bash
	#converts downloaded tar file to correct page order to make cbz
	
	INDEX=0
	
	cat manifest.json | grep src_image > tmp.txt
	sed -i 's/"//' tmp.txt 
	sed -i 's/"//' tmp.txt 
	sed -i 's/"//' tmp.txt 
	sed -i 's/"//' tmp.txt 
	sed -i 's/      src_image: //' tmp.txt 
	sed -i 's/,//' tmp.txt 
	
	while read p; do
	   printf -v filename "%04d" $INDEX
	   mv "$p" "$filename.jpg"
	   ((INDEX++))
	done <tmp.txt

	rm manifest.json
	rm manifest.jsonp
	rm tmp.txt



(So paste all that into a text file, call it "rename-thingy.txt" then change the properties to make it executable (or "chmod +x rename-thingy.txt", then remember where it was so you can run it :)


Once you have “book.tar”, unzip (yes I know untar sorry) it onto its own folder, go the folder in the terminal, and run the shell script.

All going well, it ought to rename the randomised file names to 001.jpg 002.jpg and so on.

Rezip the folder and that should be the cbz file. 

Be aware that all this the script is doing is reading through the json file and pulling out the randomised filenames in sequential order, and then renaming into that order.  The proper way to do this is reading the index pairs with a proper json reading tool like jq but… um…  this was cobbled together in an hour and works fine as it is, so it doesn’t do that.  The grep thing is the laziest minimum-effort way to get this to work. Doing this properly would be pulling the sort_order and src_image pairs from the json and building an array inside the loop to rename everything.

It has always worked for me, though.

Once I ended up with two extra un-renamed files, but looking it was obvous they were two halves of one of the special extras pages at the end of the book, and there was already a joined-together image of them - so I think it was actually a dead leftover page that wasn't visible in the real file.

No other problems beyond that.
