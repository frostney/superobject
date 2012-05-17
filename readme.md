This is a fork of [http://code.google.com/p/superobject](http://code.google.com/p/superobject)

# SuperObject #

**What is JSON ?**

JSON (JavaScript Object Notation) is a lightweight data-interchange format.
It is easy for humans to read and write. 
It is easy for machines to parse and generate. 
It is based on a subset of the JavaScript Programming Language, Standard ECMA-262 3rd Edition - December 1999. 
JSON is a text format that is completely language independent but uses conventions that are familiar to programmers.
These properties make JSON an ideal data-interchange language.
You can get more informations on [json.org](http://www.json.org).

	{
	  "name": "Henri Gourvest", /* this is a comment */
	  "vip": true,
	  "telephones": ["000000000", "111111111111"],
	  "age": 33,
	  "size": 1.83,
	  "adresses": [
	    {
	      "adress": "blabla",
	      "city": "Metz",
	      "pc": 57000
	    },
	    {
	      "adress": "blabla",
	      "city": "Nantes",
	      "pc": 44000
	    }
	  ]
	}

**Parsing a JSON data structure**

	var
	  obj: ISuperObject;
	begin
	  obj := SO('{"foo": true}');
	  obj := TSuperObject.ParseString('{"foo": true}');
	  obj := TSuperObject.ParseStream(stream);
	  obj := TSuperObject.ParseFile(FileName);
	end;

**Accessing data**

There isn't individual datastructure for each supported data types.
They are all an object: the ISuperObject.

	  val := obj.AsString;
	  val := obj.AsInteger;
	  val := obj.AsBoolean;
	  val := obj.AsDouble;
	  val := obj.AsArray;
	  val := obj.AsObject;
	  val := obj.AsMethod;

**How to read a property value of an object ?**

	  val := obj.AsObject.S['foo']; // get a string
	  val := obj.AsObject.I['foo']; // get an Int64
	  val := obj.AsObject.B['foo']; // get a Boolean
	  val := obj.AsObject.D['foo']; // get a Double
	  val := obj.AsObject.O['foo']; // get an Object (default)
	  val := obj.AsObject.M['foo']; // get a Method
	  val := obj.AsObject.N['foo']; // get a null object

**How to read a value from an array ?**

	  // the advanced way
	  val := obj.AsArray.S[0]; // get a string
	  val := obj.AsArray.I[0]; // get an Int64
	  val := obj.AsArray.B[0]; // get a Boolean
	  val := obj.AsArray.D[0]; // get a Double
	  val := obj.AsArray.O[0]; // get an Object (default)
	  val := obj.AsArray.M[0]; // get a Method
	  val := obj.AsArray.N[0]; // get a null object

**Using paths**

Using paths is a very productive method to find an object when you know where is it.
This is some usage cases:

	  obj['foo']; // get a property
	  obj['123']; // get an item array
	  obj['foo.list']; // get a property from an object
	  obj['foo[123]']; // get an item array from an object
	  obj['foo(1,2,3)']; // call a method
	  obj['foo[]'] := value; // add an item array

you also can encapsulate paths:

	  obj := so('{"index": 1, "items": ["item 1", "item 2", "item 3"]}');
	  obj['items[index]'] // return "item 2"

or recreate a new data structure from another:

	  obj := so('{"index": 1, "items": ["item 1", "item 2", "item 3"]}');
	  obj['{"item": items[index], "index": index}'] // return {"item": "item 2", "index": 1}

**Browsing data structure**

*Using Delphi enumerator.*

Using Delphi enumerator you can browse item's array or property's object value in the same maner.

	var
	  item: ISuperObject;
	begin
	  for item in obj['items'] do ...

you can also browse the keys and values of an object like this:

	var
	  item: TSuperAvlEntry;
	begin
	  for item in obj.AsObject do ...
	  begin
	    item.Name;
	    item.Value;
	  end;

*Browsing object properties without enumerator*

	var
	  item: TSuperObjectIter;
	begin
	  if ObjectFindFirst(obj, item) then
	  repeat
	    item.key;
	    item.val;
	  until not ObjectFindNext(item);
	  ObjectFindClose(item);

*Browsing array items without enumerator*

	var
	  item: Integer;
	begin
	  for item := 0 to obj.AsArray.Length - 1 do
	    obj.AsArray[item]

**RTTI & marshalling in Delphi 2010**

	type
	  TData = record
	    str: string;
	    int: Integer;
	    bool: Boolean;
	    flt: Double;
	  end;
	var
	  ctx: TSuperRttiContext;
	  data: TData;
	  obj: ISuperObject;
	begin
	  ctx := TSuperRttiContext.Create;
	  try
	    data := ctx.AsType<TData>(SO('{str: "foo", int: 123, bool: true, flt: 1.23}'));
	    obj := ctx.AsJson<TData>(data);
	  finally
	    ctx.Free;
	  end;
	end;

**Saving data**

	  obj.AsJSon(options);
	  obj.SaveTo(stream);
	  obj.SaveTo(filename);

**Helpers**

	  SO(['prop1', true, 'prop2', 123]); // return an object {"prop1": true, "prop2": 123}
	  SA([true, 123]); // return an array [true, 123]

**Non canonical forms**

The SuperObject is able to parse non canonical forms.

	// unquoted identifiers
	SO('{foo: true}');
	// unescaped or unquoted strings
	SO('{собственность: bla bla bla}');
	// excadecimal
	SO('{foo: \xFF}');
