# FileMaker Edit Text Files
This set of scripts and functions allows you to read from and write to a text file stored in a container.  It can convert between ASCII text and Base64 encoded text.  When you couple this with FileMaker's native [Base64Decode](http://www.filemaker.com/help/13/fmp/en/html/func_ref1.31.13.html) and [Base64Encode](http://www.filemaker.com/help/13/fmp/en/html/func_ref1.31.14.html) functions, you can easily edit text files stored in containers.  I developed these functions specifically to support syncing data between offline databases deployed on iPads, which do not allow plugins.

Check out the example [FileMaker file](https://github.com/jmtritch/FileMaker_Edit_Text_Files/blob/master/FileMaker_Text_File_Manipulation_Example.fmp12) to see these scripts and functions in action.

The following explains at a fairly high level how these functions work to read and write to text files.  Let's start by exploring how to write to a file stored in a container.

## Limitations
These functions rely heavily on recursion.  When converting more than a few thousand characters, the functions fail due to exceeding the [50,000 recursive calls limit](http://help.filemaker.com/app/answers/detail/a_id/11889/~/technical-specifications-of-filemaker-pro-13-and-filemaker-pro-13-advanced).  To get around this, I have added scripts that limit the recursive calls by looping through chunks of the text.  The __Ascii To Base64__ script limits the ```AsiiToBase64``` function to receive only 57 charcaters at a time as it loops through the text.  The __Base64 To Ascii__ script limits the ```Base64toAscii``` function to a single line of 76 characters.

These functions are not fast.  Since each character is converted to a text representation of its binary value during the process, it takes time to convert between ASCII and Base64.  If you are running FileMaker on Windows or Mac and can install plugins, there are better options for extracting/inserting text from/into a file.

## Writing Text to a File --> ASCII To Base64 Script

To write text to a file, use the __ASCII To Base64__ script and the native ```Base64Decode``` function.  __ASCII To Base64__ takes the ASCII text as a parameter and returns the corresponding Base64 text by utilizing the custom function ```AsciiToBase64```.  You can then insert the Base64 encoded text into a container by setting the container field with the FileMaker function, ```Base64Decode ( [ASCII_To_Base64_ScriptResult] {; [ Optional File Name ]} )```.

Let's walk through how the function ```AsciiToBase64``` works.

### AsciiToBase64

The ```AsciiToBase64``` function converts ASCII text to Base64 text.  It depends on some of the additional functions included in this repository, which are explored below. Using the classic example of ```Hello World```, we will walk through the steps of this function.

First, the function converts the ```Hello World``` ASCII text to a text representation of its binary value.

#### AsciiToBin

The ```AsciiToBin``` function loops through all of the characters of the text and converts each character to its 8-bit binary value:

```
H  -> 01001000
e  -> 01100101
l  -> 01101100
l  -> 01101100
o  -> 01101111
sp -> 01000000
W  -> 01010111
o  -> 01101111
r  -> 01010010
l  -> 01101100
d  -> 01100100
```

It does this by converting each character to its corresponding integer value using FileMaker's ```Code``` function.  Then it uses the custom function ```IntToByte``` to convert the integer value of the character to its 8-bit binary representation, and concatenates them all together sequentially.

All together, ```AsciiToBin ( "Hello World" )``` produces:
```010010000110010101101100011011000110111100100000010101110110111101110010011011000110010000100001```.

Now that we have our binary representation, we can convert it to Base64.

#### BinToBase64

This function splits the binary text into groups of six bits.  It loops through each 6-bit binary number and converts each number to its corresponding Base64 character.  All of the characters convert as follows:
```
010010 -> S
000110 -> G
010101 -> V
101100 -> s
011011 -> b
000110 -> G
111100 -> 8
100000 -> g
010101 -> V
110110 -> 2
111101 -> 9
110010 -> y
011011 -> b
000110 -> G
0100 -> ???
```
The final binary number has only four bits.  It needs to be padded with two zeros to make a 6-bit number before it can be converted to Base64.

**4 bits in last binary value**

If we have a 4-bit final binary number, like above, then we pad it with two zeros to make a 6-bit binary number.  In this case, our ```0100``` becomes ```0100000```, which converts to ```Q```.  We're not done, though, because we need to let the Base64 reader know that we padded the last two bits.  To indicate a 2-bit padding, we add the character ```=```.  Our final binary set converts as follows: ```0100 -> Q= ```.

**2 bits in last binary value**

If we have a 2-bit final binary number, then we pad it with four zeros to make a 6-bit binary number.  In this case our ```01``` becomes ```0100000``` which converts to ```Q```.  To indicate a 4-bit padding, we add two equal signs, ```==```.  This final binary number would convert as follows: ```0100 -> Q==```.

Now we have converted ```Hello World``` to binary and then to the Base64 text ```SGVsbG8gV29ybGQ=```.  This text is ready to be placed into a file using FileMaker's ```Base64Decode``` function, which will export the Base64 encoded text into a file.

## Reading Text from a File --> Base64 To ASCII Script

To extract text from a file stored in a container, use the native ```Base64Encode``` function and the __Base64 To ASCII__ script.  It takes the Base64 encoded text as a parameter and returns the corresponding ASCII text by utilizing the custom function ```Base64ToAscii```.  If you pass the script ```Base64Encode ( TableName::FileName )```, where _TableName::fileName_ is the container field with your text file, it will return the ASCII text.  You can then manipulate that text or store it in a field, as needed.

### Base64ToAscii
The ```Base64ToAscii``` function converts Base64 text to ASCII text.  It depends on some of the additional functions included in this repository. Using our Base64 encoding of Hello World, ```SGVsbG8gV29ybGQ=```, we will walk through the steps of how this function works.  First, we need to convert this Base64 text to a text representation of its binary value.

#### BinToBase64

This function simply loops through each character, and converts it to a 6-bit binary number.  It is the exact reverse of the steps above, and we get ```010010000110010101101100011011000110111100100000010101110110111101110010011011000110010000100001```.

#### BinToAscii
This function splits the binary text into groups of eight bits, also known as a byte.  It loops through each byte, and converts the byte to its corresponding ASCII character.  It does this by converting each byte to its corresponding integer value using ```ByteToInt```, and FileMaker's ```Char``` function to convert the integer to the corresponding ASCII character.

## Final Thoughts
If you want native support for editing files within FileMaker, then these functions are all you need.  They will enable you to edit text files with the push of a button.  Please checkout the working example to see these functions in action.  I'm always looking for feedback, so let me know if find any problems or have any recommendations for improvements.
