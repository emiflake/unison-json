So you want to decode JSON from the outside world huh, that's pretty scary!
But by using some simple patterns, it doesn't have to be.
In this short tutorial, we'll cover how to decode a simple API response that lists a bunch of users with their friends, and we'll go over how to model it.

## Let's start with taking a look at the entire endpoint that we will model.

```json
{
  "total": 2,
  "users": [
      {
        "id": 0,
        "name": "Admin",
        "likes": [ "cooking", "managing" ]
      },
      {
      "id": 1,
      "name": "Alice",
      "likes": [ "cooking", "programming" ]
      }
  ]
}
```

Okay okay, I know this looks pretty large, but let's start small!
We could for instance start with just parsing the response total.
We can do this by extracting the "total" field.


```hs
Decode.field "total" Decode.int
```

This will already work on our endpoint, and will successfully give us an int! Use it like so:

```hs
Decode.runParse (Decode.field "total" Decode.int) endpointText
```

Which will return `(Right +2)`

## Okay, but how do we combine this? What if we have multiple fields?

Well, let's see if we can try to model a user, ignoring what they like for now.


```hs
type User = {
  id : Int,
  name : Text
}
```

This creates a user type, and we can construct a user by using the `User` constructor.

```hs
User +1 "Alice"
```

Helpfully, we can follow the applicative pattern to construct a decoder for our user type.

```hs
decodeUser : Decode User
decodeUser =
  Decode.succeed User
    <*> (Decode.field "id" Decode.int)
    <*> (Decode.field "name" Decode.string)
```

Generally, the pattern is always more or less the same.

First, we have the type we want to create passed into @Decode.succeed

```hs
Decode.succeed YourType
```


Then, we have each of the fields you want to feed into the type.


```hs
    <*> (Decode.field "myField" myFieldDecoder)
    <*> (Decode.field "myField2" myOtherFieldDecoder)
```

In our case, we could use @Decode.int and @Decode.string, because our fields contained
those primitive types. But if you have nesting, you can put your own decoders in there!

## So, we decoded the ID and the name, but what about the likes?

Well, we could say that the "likes" is a list of strings...

```hs
type User = {
  id : Int,
  name : Text,
  likes : [String]
}
```

But, if we know what kinds of things we can like, we could put that into a special type, which
would be much safer!

```hs
type Hobby = Cooking | Managing | Programming
```

Now, our list of strings, would be a list of `Hobby`s

```hs
type User = {
  id : Int,
  name : Text,
  likes : [Hobby]
}
```

Okay, but how can we convert from a string into a @TUTORIAL.Hobby?

Well, we have three functions that together can be used to create this relation:

```
decodeHobby : Decode Hobby
decodeHobby =
  Decode.oneOf
    [ Decode.s "cooking" ==> Cooking
    , Decode.s "managing" ==> Managing
    , Decode.s "programming" ==> Programming
    ]
```


As you can see, it's pretty clear how that relation is layed out. Whenever we encounter "cooking" we'll map that to the constructor Cooking, etc.

And, it has to be one of those, or else it will error out.

So, our updated decodeUser will look like this:

```hs
decodeUser : Decode User
decodeUser =
  Decode.succeed User
    <*> (Decode.field "id" Decode.int)
    <*> (Decode.field "name" Decode.string)
    <*> (Decode.field "likes" (@ecode.array decodeHobby))
```


Look in the TUTORIAL namespace for supporting code.
