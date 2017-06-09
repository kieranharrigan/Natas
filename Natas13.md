# NATAS: Level 13 -> Level 14
**Goal:** Getting the password for Natas14 requires that we somehow manipulate the files we upload in order to run arbitrary commands. If we can run any command on the server that we want, finding the password should be simple. Unlike Natas13, this level performs checks to make sure the file we upload is actually an image.

**Solution:**
1. Click the [View sourcecode](http://natas13.natas.labs.overthewire.org/index-source.html) link.

   We notice that the way Natas13 checks for images is via the exif_imagetype() method: `exif_imagetype($_FILES['uploadedfile']['tmp_name'])`

   exif_imagetype() is an interesting way to check because all it does it look to see if a file has the proper image signature. These signatures can easily be googled for and therefore easily faked.
2. Let's take jpeg for example, if the first 4 bytes of a file are `FF D8 FF E0` then the file is a jpeg.

   Knowing this, let's create our own "jpeg" image. Open up your shell of choice and run the following command: `echo -e "\xff\xd8\xff\xe0" > fakejpg`

   The file extension is not necessary since we'll tell the server that our file extension is .php later on.
3. Open your new fake jpeg in your favorite text editor, it's time to add our malicious php script.

   Ignore the random characters at the start of the file (these are the hex bytes we just wrote) and paste the following on the next line: `<?php echo exec("cat /etc/natas_webpass/natas14"); ?>`
4. Make sure to save your fake jpeg file and now head back to the Natas13 main page.

   Using Chrome's Developer Tools, right-click on the hidden input with the name "filename" and select "Edit as HTML".

   Change where it says .jpg to .php.

   Now the server will be serving our uploaded file as a php script.
5. Choose the file you just saved and hit "Upload File".

   You will be redirected to a page that will say something similar to: `The file upload/1nqi0bnmog.php has been uploaded`

   Click on the link and you'll see 4 random characters (our jpeg signature) followed by the password.