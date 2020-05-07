# unison-json

An implementation of JSON decoders inspired by elm/json

## Usage

Pull the latest release using

```
.> pull https://github.com/emiflake/unison-json.git:.releases._v0 external.json.v0
```

This library provides `Decode a` and ways to parse JSON values into Unison values.

Look at [TUTORIAL.md](https://github.com/emiflake/unison-json/blob/master/TUTORIAL.md) for an example of how to use it.


## How stable is it right now?

Well, I have yet to spot any huge bugs. The performance is quite poor, but it is able to parse small-ish structure pretty okay. Useful in combination with a HTTP client maybe.
The error messages are quite pleasant for Decode errors, not so much for Parse errors.
