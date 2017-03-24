# FileMaker Edit Text Files
This set of functions converts between ASCII and Base64 encoding.  When you couple these functions with FileMaker's native Base64Decode and Base64Encode functions, you can easily edit text files stored in containers.  I developed these functions specifically to support the iOS version of FileMaker, which does not allow for plugins.

Check out the example FileMaker file to see how these functions in action.  The functions all rely pretty heavily on recursion since FileMaker does not allow looping within functions.

The following explains at a fairly high level how these functions work to convert between ASCII and Base64.  Let's start by exploring how text can be exported to a file stored in a container.

## Exporting Text

To export text from a field or variable to a container you only need a single calculation with two functions: 

```
Base64Decode ( AsciiToBase64 ( [Text_Field_Name] ) { ; [Optional_file_name] } )
```

```AsciiToBase64``` converts the ASCII text to Base64 encoded text, and FileMaker's native function, ```Base64Decode``` converts that text back to true binary and sets it in the container specified in the calculation.  [Wikipedia](https://en.wikipedia.org/wiki/Base64) has a pretty good article explaining Base64 encoding.

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

## Importing Text

To import text from a container into a text field you only need a single calculation with two functions:

```
Base64ToAscii ( Base64Encode ( [Container_Name] ) )
```

```Base64Encode``` is a native FileMaker function that converts the file data in the provided container to Base64 encoding.  Base64 encoding essentially converts six binary bits to a character.  

Now that our data is encoded as Base64 text, the custom function ```Base64ToAscii``` converts that text to ASCII text.

### Base64ToAscii
The ```Base64ToAscii``` function converts Base64 text to ASCII text.  It depends on some of the additional functions included in this repository. Using our Base64 encoding of Hello World, ```SGVsbG8gV29ybGQ=```, we will walk through the steps of how this function works.  First, we need to convert this Base64 text to a text representation of its binary value.

#### BinToBase64

This function simply loops through each character, and converts it to a 6-bit binary number.  It is the exact reverse of the steps above, and we get ```010010000110010101101100011011000110111100100000010101110110111101110010011011000110010000100001```.

#### BinToAscii
This function splits the binary text into groups of eight bits, also known as a byte.  It loops through each byte, and converts the byte to its corresponding ASCII character.  It does this by converting each byte to its corresponding integer value using ```ByteToInt```, and FileMaker's ```Char``` function to convert the integer to the corresponding ASCII character.

## Final Thoughts
If you want native support for editing files within FileMaker, then these functions are all you need.  They will enable you to edit text files with the push of a button.  Please checkout the working example to see these functions in action.  I'm always looking for feedback, so let me know if find any problems or have any recommendations for improvements.
