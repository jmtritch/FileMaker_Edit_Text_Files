# FileMaker Text File Manipulation
This set of functions allows for FileMaker to edit text files natively within FileMaker.  I developed these functions specifically to support the iOS version of FileMaker.  With these functions you can import ASCII text from a file within a container and you can export text from within FileMaker to a text file stored in a container.

Check out the functions to see how the code works.  I tried to make ensure the code is well commented, but please let me know if you have any questions or comments.

## Importing Text

To import text from a container into a text field you only need a single calculation with two functions:

```
Base64ToAscii ( Base64Encode ( [Container_Name] ) )
```

```Base64Encode``` is a native FileMaker function that converts the file data in the provided container to base64 encoding.  Base64 encoding essentially converts six binary bits to a character.  [Wikipedia](https://en.wikipedia.org/wiki/Base64) has a pretty good article showing how the 6 bits converts to the corresponding character.

Now that our data is encoded as base64 text, the custom function ```Base64ToAscii``` converts that text to ASCII text.

## Exporting Text

To export text from a field to a container you also only need a single calculation with two functions: 

```
Base64Decode ( AsciiToBase64 ( [Text_Field_Name] ) { ; Optional_file_name } )
```

```AsciiToBase64``` converts the ASCII text to base64 encoded text.

Now we can use FileMaker's native function, ```Base64Decode``` to converts that data back to true binary and set it in our container.

## Base64ToAscii Function

The ```Base64ToAscii``` function depends on some of the additional functions included in this repository.
