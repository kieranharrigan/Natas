# NATAS: Level 11 -> Level 12
**Goal:** Getting the password for Natas12 relies on a combination of cookie hijacking and one-time pad decoding. Natas11 provides us with a "data" cookie upon arrival to the page. Viewing the sourcecode via the [View sourcecode](http://natas11.natas.labs.overthewire.org/index-source.html) link shows us that the data cookie is an XOR encrypted version of a JSON string. The xor_encrypt() function itself is offered to us in the sourcecode as well, but the one-time pad key ($key) has been censored out. Without knowing the key, the solution won't be as easy as just passing the data cookie back into xor_encrypt() to reverse the encryption.

**Solution:**
1. In Chrome, pop open dev tools and click on the Application tab.

   View the cookies for Natas11 and get the value of "data". The default should be exactly:
   `ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw%3D`

   If you've messed around with background color at all the cookie will look slightly different.
   Make sure you have this cookie on hand because we'll need it later.
2. Click the [View sourcecode](http://natas11.natas.labs.overthewire.org/index-source.html) link.

   We notice that the default JSON structure for the data cookie is as follows:
   `{"showpassword":"no","bgcolor":"#ffffff"}`

   There is an if block towards the bottom that will print us the Natas12 secret so long as "showpassword" is mapped to "yes" in the JSON string.

   So, we have to modify our data cookie to represent an encrypted version of the JSON string with "yes" rather than "no".
3. If we knew the one-time pad key used to encrypt the original JSON, then we could successfully encrypt our modified JSON too.

   We know the default JSON structure that resulted in our default data cookie. This means we should be able to XOR each character in the default JSON string against all 256 ASCII characters until we find the result that matches each character in our cookie (at the same index).

   The following is a simple brute force php script run against the default JSON string and default data cookie:
   ```
    <?php
    function xor_encrypt() {
       $cookie = base64_decode('ClVLIh4ASCsCBE8lAxMacFMZV2hdVVotEhhUJQNVAmhSEV4sFxFeaAw%3D');
       $json = '{"showpassword":"no","bgcolor":"#ffffff"}';

       $outText = '';

       for ($i = 0; $i < strlen($cookie); $i++) {
          for ($j = 0; $j < 255; $j++) {
             $xor = $json[$i] ^ chr($j);

             if ($xor === $cookie[$i]) {
               echo chr($j);
             }
          }
       }
    }
    
    xor_encrypt();
    echo PHP_EOL;
    ?>
    ```

   The resulting output should be: `qw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jqw8Jq`
   Right away we notice that `qw8J` simply repeats itself (this is standard when a one-time pad key's length doesn't match the length of the string you're encrypting).

   This means that `qw8J` is the key that xor_encrypt() uses!
4. Now that we know the key, we can correctly encrypt whatever we'd like–including a JSON string that says to show us the password.

   Here is the final php script used to generate our encrypted "unlock" cookie:
   ```
   <?php
    function xor_encrypt($in) {
       $key = 'qw8J';
       $text = $in;
       $outText = '';

       // Iterate through each character
       for ($i = 0; $i < strlen($text); $i++) {
          $outText .= $text[$i] ^ $key[$i % strlen($key)];
       }

       return $outText;
    }

    $unlock = array("showpassword" => "yes", "bgcolor" => "#ffffff");
    echo base64_encode(xor_encrypt(json_encode($unlock))) . PHP_EOL;
    ?>
    ```

   After running the script we get: `ClVLIh4ASCsCBE8lAxMacFMOXTlTWxooFhRXJh4FGnBTVF4sFxFeLFMK`
   This is our solution cookie! Time to go test it out.
5. Head back over to the cookies for Natas11 in Chrome.

   In the value field for "data", replace the string with the one we just generated.

   Refresh the page and boom, the Natas12 secret is revealed.