eson
===

eson (extensible *single* object notation) is a language for declarative programming. eson is built on top of JSON which makes it easy for anyone familiar with JSON to read, write and understand eson programs. eson and it's compiler can be used for simple JSON preprocessing or as the base syntax and environment for declarative external DSLs. 

#Syntax
eson programs use valid JSON syntax and conform to the [JSON format](http://json.org/), JSON structures such as objects, pairs and values are specific forms of their equivalents in eson. eson extends the semantic expression of JSON, adding programmable declarativeness to the data notation JSON provides. eson programs are made up of unordered declarations, the three types of declaration are attributes, calls and lets.

##Attributes
The attribute is a regular JSON name/vaule pair. The eson reader accepts all JSON pairs without prefixes of '&' and '&' as variable assignment operations. Where the key/name is the identifier name and the value is the data bound to the variable. The identifier can be referenced in the program by the `$` prefix.

##Special forms
Special forms are procedures/functions in the eson reader. In compliance with eson's declarative programming style, special forms satisfy properties of the program upon evaluation. This is in contrast to executing commands or a sequences of commands in an imperative programming style. All special forms guarantee *referential transparency*.

###let
the let special form performs variable creation which allows the use of unbounded variables in eson programs. Let takes an array of JSON strings and creates unbound variables for each element of the array. let can only be called once per program and should appear as a call as it evaluates to null. An unbound or bound variable is referenced by the `$` prefix in the program.

```JSON
"&let":["red","blue"]
```

The snippet above creates two variables `$red` and `$blue`.

###ref
ref takes a [JSON pointer](https://tools.ietf.org/html/rfc6901) and returns the specified JSON value.

```JSON
{ "&ref": "#/foo" }
```

###doc
Generate a doc pair for a given keyword and add it to a nested object for the keyword pair. In the example below `int_eson` and `int_json` are shown in the same document for brevity.

```JSON
{
  "int_eson": 90,
  "&&doc": [
    [
      "int_eson",
      "JSON can have integers"
    ]
  ],
  "int_json": {
    "value": 90,
    "doc": "JSON can have integers"
  }
}
```

##Singles and their calls
A call is an eson primitive for performing procedure application. A call is a JSON pair which allows procedures defined in the eson reader to be evaluated in the eson program. Because JSON objects are not recursive data structures, procedure application takes two forms: one supporting evaluation only and another form where both evaluation and substitution can take place. 

A call is a JSON pair which satisfies the following conditions:

1. key MUST be prefixed with `&`,
1. string following the `&` prefix MUST be the name of a special form,
1. and value MUST be either an array, null or a single. 

```JSON
{ 
  "name1": "value",
  "&special-form": ["param"]
}
```

If an array is provided as the value, it's elements are passed as parameters to the procedure. The reader interprets a single with a null value as a 0-arity function call. If the length of the array does not match the arity expected by the function then an error is thrown. When the reader interprets a JSON member as a call it first evaluates it and then removes the member from their current position in the eson document. An eson reader must retain these members in a special array with the keyword "env". The reader should allow the user to choose whether or not the "env" member should be returned. 

A single performs procedure application with substitution, allowing for *referential transparency*. eson defines a single as a special JSON object containing a call; after the single is evaluated the single (the JSON object) is substituted with a JSON value. Please note that the *referential transparency* is a guarantee of the underlying procedure and not the single. A call is sometimes referred to as a *weak single* because it does not provide useful referential transparency. It provides referential transparency in the sense that the single will be replaced by the "empty member" but this is not a useful result for declarative combinination. Because duplicate keys are not legal JSON, a call for a special form or procedure can only be defined once per program. Since singles appear as JSON values, singles can be used any time a JSON key is defined.

A single is a JSON object which satisfies the following conditions:

1. MUST contain a call,
2. and MUST contain only one JSON pair.

```JSON
{
  "name1": {"&symbol1": ["param"]},
  "name2": {"&symbol2": null}
}
```

Singles get their name as a result of being a 1-tuple JSON object. A JSON object is unordered and does not normally qualify as a tuple but the 1 element unordered collection is equivalent to the 1 element ordered collection. 

###Single or call?
There exists a special case where an eson program consists of a single call. The question this raises is whether this is interpreted as a single or a call? It's actually both and neither.

```JSON
{
  "&special-form": ["param"]
}
```

The eson reader will treat this as a single where the call is evaluated but no substitution is possible because the resulting JSON document is the empty document `{}` and no substitution can take place within the empty document. Thus the observed result is that of a call and not a single. This illustrates that the single is just a regular eson program. In this scenario the eson reader should return the value of the call instead of nil.


