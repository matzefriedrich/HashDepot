# HashDepot
[![NuGet Version](https://img.shields.io/nuget/v/HashDepot.svg)](https://www.nuget.org/packages/HashDepot/)
[![Build Status](https://travis-ci.org/ssg/HashDepot.svg?branch=master)](https://travis-ci.org/ssg/HashDepot)

I have been implementing various hash functions that are absent in .NET framework. 
I decided to converge them into a library. My primary goals are to provide well-tested and 
performant implementations. The library currently supports [SipHash](https://131002.net/siphash/),
[MurmurHash3](https://en.wikipedia.org/wiki/MurmurHash), [xxHash](http://cyan4973.github.io/xxHash/)
and [FNV-1a](https://en.wikipedia.org/wiki/Fowler%E2%80%93Noll%E2%80%93Vo_hash_function#FNV-1a_hash). 

To install it on NuGet:

    Install-Package HashDepot

# Supported Hash Algorithms
I try to add anything that's not in C# runtime and quite popular. For instance,
there are multiple xxHash implementations for C# but they differentiate in terms of API
complexity and performance. Although I didn't try out SIMD optimizations, the existing code
is quite fast.

## xxHash
This one claims to be one of the fastest hash functions and it's actually amazing. Even without any SIMD
optimizations, it outperforms everything, even a plain checksum by a factor of two. The implementation
assumes little endian machines. Example usage:

```csharp
var buffer = Encoding.UTF8.GetBytes("some string");
uint result = XXHash.Hash32(buffer, seed: 123);
ulong result = XXHash.Hash64(buffer); // default seed is zero
```

## SipHash
SipHash is resistant to hash-flood attacks against hashtables and uses
a key parameter to ensure HMAC-like authenticity yet faster. Unfortuantely a native 
.NET implementation does not exist. It is my take on it, and it is really fast for a 
managed environment. It's standard SipHash-2-4 implementation with 64-bit. To use it:

```csharp
var buffer = Encoding.UTF8.GetBytes("some string");
var key = new byte[16] { .. your random key here .. };
ulong result = SipHash24.Hash64(buffer, key);
```

If you have a larger buffer than 2GB it's better to use streaming functions instead.

## MurmurHash3
MurmurHash3 provides a good balance between performance and homogenity but is 
essentially prone to hash-flood attacks (trivial to force collisions). HashDepot
implements its x86 flavor (not x64). An example use is:

```csharp
var buffer = Encoding.UTF8.GetBytes("some string");
uint seed = // .. preferred seed value ...
uint result = MurmurHash3.Hash32(buffer, seed);
```

## FNV
A straightforward implementation of FNV-1 and FNV-1a hash algorithm for .NET. Usage is 
very simple. For instance to calculate 32-bit FNV-1a hash of ASCII string "some string":

```csharp
var buffer = Encoding.UTF8.GetBytes("some string");
uint result = Fnv1a.Hash32(buffer); // 32-bit hash
ulong result = Fnv1a.Hash64(buffer); // 64-bit hash
```
  
## Streaming and Async functions
All hashes also provide stream-based (albeit slow) functions with their async variants too. In order to
get the hash of a stream just call the function with a stream instead of a memory buffer:

```csharp
ulong result = XXHash.Hash64(stream);
```

If you'd like to run it asynchronously, use the async variant:

```csharp
uint result = await MurmurHash3.Hash32Async(stream);
```

# Benchmarks

BenchmarkDotNet=v0.13.2, OS=Windows 11 (10.0.22621.819)
Snapdragon Compute Platform, 1 CPU, 8 logical and 8 physical cores
.NET SDK=7.0.100
  [Host]     : .NET 6.0.11 (6.0.1122.52304), Arm64 RyuJIT AdvSIMD
  DefaultJob : .NET 6.0.11 (6.0.1122.52304), Arm64 RyuJIT AdvSIMD


|          Method |       Mean |   Error |  StdDev |
|---------------- |-----------:|--------:|--------:|
|     Checksum_32 |   238.2 us | 1.03 us | 0.92 us |
|       XXHash_32 |   254.4 us | 1.92 us | 1.70 us |
|       XXHash_64 |   242.2 us | 1.02 us | 0.96 us |
| MurmurHash3_x86 |   483.9 us | 2.64 us | 2.34 us |
|    SipHash24_32 |   523.6 us | 3.57 us | 3.34 us |
|        Fnv1a_32 | 1,109.8 us | 5.72 us | 5.07 us |
|        Fnv1a_64 | 1,123.8 us | 5.89 us | 5.51 us |

# Contributing
You're more than welcome to contribute fixes or new hash algorithms. Please keep these in mind:

- Make sure the code builds without warnings.
- Include unit tests for the fixed bug, or the new feature.
- If you're proposing a new hash algorithm, please make sure that it's not in C# runtime, there isn't an
  existing library that is already tremendously popular, and HashDepot's simplistic approach would provide
  a benefit over the alternatives.

# License
MIT License. See LICENSE file for details
