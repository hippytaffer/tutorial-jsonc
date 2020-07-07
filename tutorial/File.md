# Reading a JSON file

Rather than start with the problem of how to parse a JSON file, let's aproach the subject rather slowly. First, we consider how to read a JSON file into memory and print the file contents as a c string. For this example and many of the rest of the examples, I am going to use the JSON representation of a person taken from the aforementioned [Wikipedia article](https://en.wikipedia.org/wiki/JSON). Copy and paste the JSON below into a text editor and save as _*contact.json*_:

```
{
  "firstName": "John",
  "lastName": "Smith",
  "isAlive": true,
  "age": 27,
  "address": {
    "streetAddress": "21 2nd Street",
    "city": "New York",
    "state": "NY",
    "postalCode": "10021-3100"
  },
  "phoneNumbers": [
    {
      "type": "home",
      "number": "212 555-1234"
    },
    {
      "type": "office",
      "number": "646 555-4567"
    }
  ],
  "children": [],
  "spouse": null
}

```
Now in the same directory as contact.json create the file _*json-file00.c*_ with the below contents:

```
#include <stdio.h>
#include <json-c/json.h>

int
main(void)
{
   json_object *root = json_object_from_file("contact.json");
   if (!root)
      return 0;
   printf("The json file:\n\n%s\n", json_object_to_json_string(root));
   json_object_put(root);
   return 1;
}

```
Compile with:

```
 gcc json-file00.c -ljson-c -o json-file00
```

Now run _*json-file00*_:

```
$ ./json-file00
The json file:

{ "firstName": "John", "lastName": "Smith", "isAlive": true, "age": 27, "address": { "streetAddress": "21 2nd Street", "city": "New York", "state": "NY", "postalCode": "10021-3100" }, "phoneNumbers": [ { "type": "home", "number": "212 555-1234" }, { "type": "office", "number": "646 555-4567" } ], "children": [ ], "spouse": null }

```

Admitttedly, the output is not very pretty, all extraneous whitepace has been stripped from the document. That may or may not be what you want in your application. Fortunately, json-c offers a little more control over the printing of the JSON object.

As before, in the same directory as _*contact.json*_ create the file _*json-file01.c*_ with the below contents and compile with
_*gcc json-file01.c -ljson-c -o json-file01*_:

```
#include <stdio.h>
#include <json-c/json.h>

int
main(void)
{
   json_object *root = json_object_from_file("contact.json");
   if (!root)
      return 0;
   printf("The json file:\n\n%s\n", json_object_to_json_string_ext(root, JSON_C_TO_STRING_PRETTY));
   json_object_put(root);
   return 1;
}
```

The output from running _*json-file01*_ is:

```
$ ./json-file01
The json file:

{
  "firstName":"John",
  "lastName":"Smith",
  "isAlive":true,
  "age":27,
  "address":{
    "streetAddress":"21 2nd Street",
    "city":"New York",
    "state":"NY",
    "postalCode":"10021-3100"
  },
  "phoneNumbers":[
    {
      "type":"home",
      "number":"212 555-1234"
    },
    {
      "type":"office",
      "number":"646 555-4567"
    }
  ],
  "children":[
  ],
  "spouse":null
}

```

Here, the output is more readable having been reformatted somewhat, but it is certainly more suitable for human reading than before. Especially for more complex and long json files.

So let's examine what we have learned from the code above.

First and foremost, I have introduced a new 'type': _*json_object*_. As expected for a C program, _*json_object*_ is really a structure defined in the _*json_object_private.h*_ header file. In Ubuntu/debian distos it is installed at /usr/include/json-c. You can examine it's defintion if you are so inclined, but the details of it's implementation are not important here: we can just think of it as a basic object that the json-c functions operate upon.

Some programmers who wish to stress the idea _*json_object*_ is a *struct* would write the line of code

```
json_object *root = json_object_from_file("contact.json");
```

as

```
struct json_object *root = json_object_from_file("contact.json");
```

That is a personal choice and do whichever you prefer. I prefer to leave the struct part off and think of it more like an additional basic type the library offers, much like *int* or *float*.


Next I introduce the function:

- json_object\* *json_object_from_file*(const char \*filename)

This function should be self explanatory: It reads the file *filename* and returns the _*json_object*_ respresenting the JSON data contained in the file. If there is a failure to read the file, this function returns NULL. Of course, as we will see later json-c can also create a _*json_object*_ from a string (technically a NULL terminated character array in C, not that I am pedantic enough to care).

Now note, right before our program exits we have the function:

- void json_object_put(json_object \* obj)

We call this function on the json_object we created using *json_object_from_file*. You can think of this as freeing the memory allocated by creating the json_object. The offical documentation states _*json_object_put*_

> Decrements the reference count of json\_object and frees if it reaches zero. You must have ownership of obj prior to doing this or you will cause an imbalance in the reference count.

You may be thinking, "WTF is ownership of an object?" More on that later (hopefully), just ignore it for now and note that in this case, we need it for the *root* _*json_object*_.

Finally, I introduced 2 functions and a constant to convert the json_object to a string:

- const char\* json_object_to_json_string(json_object \*obj)
- const char\* json_object_to_json_string_ext(json_object \*obj, int flags)
- JSON_C_TO_STRING_PRETTY

First, the constant _*JSON_C_TO_STRING_PRETTY*_ used in the function _*json_object_to_json_string_ext*_ as a _foramtting flag_. There are 3 such flags used in this function, the other two are:

- JSON_C_TO_STRING_PLAIN
- JSON_C_TO_STRING_SPACED

These flags tell the function _*json_object_to_json_string_ext*_ how to format the JSON in the string representation. We have already seen the usage and effect of _*JSON_C_TO_STRING_PLAIN*_: The function _*json_object_to_json_string(obj)*_ is equivalent to _*json_object_to_json_string_ext(obj, JSON_C_TO_STRING_SPACED)*_. Here all superfluous white space is removed from the string representation.

Now, to see the effect of using JSON_C_TO_STRING_SPACED edit the file json-file01.c and add it as the flag instead of _*JSON_C_TO_STRING_PRETTY*_. What do think it does?

## Problems

1. What happens if our json file contains invalid json and we run _*json-file01*_ ? Alter the program to print a message informing the user the json is invalid.

2. todo JSON_C_TO_STRING_PRETTY | other values what does this do? I saw it in a sample program online. explore this
