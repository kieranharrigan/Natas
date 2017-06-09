# NATAS: Level 12 -> Level 13
**Goal:** Getting the password for Natas13 requires that we somehow manipulate the files we upload in order to run arbitrary commands. If we can run any command on the server that we want, finding the password should be simple.

**Solution:**
1. Click the [View sourcecode](http://natas12.natas.labs.overthewire.org/index-source.html) link.

   We notice that at some point we read in a POST variable named "filename".

   "filename" gets passed on to the makeRandomPathFromFilename() method where the extension of the file is extracted and passed to makeRandomPath().

   makeRandomPath() blindly accepts whatever extension is passed to it — THIS IS VERY IMPORTANT. This means that any extension, even scripting language extensions are allowed.
2. If we inspect the source of the main Natas13 webpage there is an important hidden input: `<input type="hidden" name="filename" value="<random_characters>.jpg">`

   This hidden input's value is what gets passed along as "filename" via POST.

   Typically, the site attempts to restrict us to only .jpg files. However, the sourcecode will accept any file type — so let's change it.
3. Using Chrome's Developer Tools, right-click on the hidden input and select "Edit as HTML".

   Change where it says .jpg to .php.

   Now the server will be serving our uploaded file as a php script.
4. This is great and all, but we still need to work out what to put in our file upload so that we can grab the password.
   
   Pop open your favorite text editor and paste in the following code:

   `<? php echo exec("cat /etc/natas_webpass/natas13"); ?>`

   This line of php targets the `/etc/natas_webpass/natas13` file. As we know, each server has a password file stored locally, but it can only be accessed by the level's account (natas13) or the previous level's account (natas12). We're on Natas12, so the server we're running commands on will have proper permissions to read from that password file.
5. You can save the php script with whatever file extension you'd like, the backend only cares about the extension we provide through "filename".
   
   Back on Natas12, choose the file you just saved and hit "Upload File".

   You will be redirected to a page that will say something similar to: `The file upload/jmcvsuu7zj.php has been uploaded`

   Click on the link and voila, the password should be echoed onto the page.