# BlobHandles
_"Blob Handles"_ are a fast & easy way to hash and compare segments of memory in C# / Unity.

They allow you to do two main things:
1) Use a sequence of bytes as a hash key, like in a `Dictionary<BlobHandle, T>`.
2) Quickly compare two handles' slices of memory for equality

## Blob Strings
This also includes _BlobString_, a wrapper around `BlobHandle` that points to an unmanaged representation of a string. 

_BlobString_ is designed for use cases that involve reading strings from an unmanaged source (network / disk) & comparing them against some sort of hash set.  

## Dictionaries

Any `Dictionary<BlobHandle, T>` gets an extension method `TryGetValueFromBytes<T>` that allows using a segment of bytes as the key to a dictionary value search, without having to construct a `BlobHandle` yourself.

For dealing with strings, there is `BlobStringDictionary<T>`.  You add regular C# strings and it takes care of conversion to the unmanaged representation for you.

## Performance Details

###### Memory & Constructors
`BlobHandle` is an immutable struct with only a pointer & a length.  This means
- fast to create
- uses little memory, no heap

###### Equality Testing
I tested a number of different ways of testing memory equality between handles to find out what method would be faster, and how that changed depending on the number of bytes to compare & the compiler (Mono or IL2CPP) used. 

Consistently the fastest method under both runtimes, easily several times faster than anything else (at least on Windows x64 where i've tested) is a direct wrapper around [memcmp](https://docs.microsoft.com/en-us/cpp/c-runtime-library/reference/memcmp-wmemcmp?view=vs-2019).  Other platforms may call for a different implementation of the equality check if it proves faster there.

###### GetHashCode()
The included method for getting a blob handle's hash code uses the length of the blob and the value of the last byte in its contents.
 
`Length * 397 ^ Pointer[Length - 1];`  

This was the fastest method I tested on my data, and it should work well on any data that doesn't have a lot of entries of the same length that _also_ end in the same byte.  

You may be able to get better performance with a different method, especially if your data is different.

###### Tests
Performance tests for BlobHandles include, but are not limited to:
- `.Equals()` performance vs string
- `.GetHashCode()` performance vs string
- `.TryGetValue(str, out T value)` performance, for `Dictionary<BlobHandle, T>` vs `Dictionary<string, T>` 
    This one is probably the most direct comparison of hash lookup performance
- `Dictionary<BlobHandle, T>.TryGetValueFromBytes()` performance, basically the same as the above test but adding the step of reading from unknown bytes like in actual use.

Performance tests are in a runtime assembly so they can be run in players.



