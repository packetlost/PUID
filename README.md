# Puid

Simple, fast, flexible and efficient generation of probably unique identifiers (`puid`, aka random strings) of intuitively specified entropy using pre-defined or custom characters.

```elixir
iex> defmodule(RandId, do: use(Puid, chars: :alpha, total: 1.0e5, risk: 1.0e12))
iex> RandId.generate()
"YAwrpLRqXGlny"
```

[![Hex Version](https://img.shields.io/hexpm/v/puid.svg "Hex Version")](https://hex.pm/packages/puid) &nbsp; [![License: MIT](https://img.shields.io/npm/l/express.svg)]()

## <a name="TOC"></a>TOC

- [Overview](#Overview)
- [Usage](#Usage)
- [Installation](#Installation)
- [Module API](#ModuleAPI)
- [Characters](#Characters)
- [Comparisons](#Comparisons)
  - [Common Solution](#Common_Solution)
  - [gen_reference](#gen_reference)
  - [misc_random](#misc_random)
  - [nanoid](#nanoid)
  - [Randomizer](#Randomizer)
  - [rand_str](#rand_str)
  - [SecureRandom](#SecureRandom)
  - [ulid](#ulid)
  - [UUID](#UUID)

## <a name="Overview"></a>Overview

**Puid** provides a means to create modules for generating random IDs. Specifically, **Puid** allows full control over all three key characteristics of generating random strings: entropy source, ID characters and ID randomness. 

A [general overview](https://github.com/puid/.github/blob/2381099d7f92bda47c35e8b5ae1085119f2a919c/profile/README.md) provides information relevant to the use of **Puid** for random IDs.

[TOC](#TOC)

### <a name="Usage"></a>Usage

`Puid` is used to create individual modules for random ID generation. Creating a random ID generator module is a simple as:

```elixir
iex> defmodule(SessionId, do: use(Puid))
iex> SessionId.generate()
"8nGA2UaIfaawX-Og61go5A"
```

The code above use default parameters, so `Puid` creates a module suitable for generating session IDs (ID entropy for the default module is 132 bits). Options allow easy and complete control of all three of the important facets of ID generation.

**Entropy Source**

`Puid` uses [:crypto.strong_rand_bytes/1](https://www.erlang.org/doc/man/crypto.html#strong_rand_bytes-1) as the default entropy source. The `rand_bytes` option can be used to specify any function of the form `(non_neg_integer) -> binary` as the source:

```elixir
iex > defmodule(PrngId, do: use(Puid, rand_bytes: &:rand.bytes/1))
iex> PrngId.generate()
"bIkrSeU6Yr8_1WHGvO0H3M"
```

**Characters**

By default, `Puid` use the [RFC 4648](https://tools.ietf.org/html/rfc4648#section-5) file system & URL safe characters. The `chars` option can by used to specify any of 16 [pre-defined character sets](#Chars) or custom characters, including Unicode:

```elixir
iex> defmodule(HexId, do: use(Puid, chars: :hex))
iex> HexId.generate()
"13fb81e35cb89e5daa5649802ad4bbbd"

iex> defmodule(DingoskyId, do: use(Puid, chars: "dingosky"))
iex> DingoskyId.generate()
"yiidgidnygkgydkodggysonydodndsnkgksgonisnko"

iex> defmodule(DingoskyUnicodeId, do: use(Puid, chars: "dîñgø$kyDÎÑGØßK¥", total: 2.5e6, risk: 1.0e15))
iex> DingoskyUnicodeId.generate()
"øßK$ggKñø$dyGîñdyØøØÎîk"

```

**Captured Entropy**

Generated IDs have at least 128-bit entropy by default. `Puid` provides a simple, intuitive way to specify ID randomness by declaring a `total` number of possible IDs with a specified `risk` of a repeat in that many IDs:

To generate up to _10 million_ random IDs with _1 in a trillion_ chance of repeat:

```elixir
iex> defmodule(MyPuid, do: use(Puid, total: 10.0e6, risk: 1.0e15))
iex> MyPuid.generate()
"T0bFZadxBYVKs5lA"
```

The `bits` option can be used to directly specify an amount of ID randomness:

```elixir
iex> defmodule(Token, do: use(Puid, bits: 256, chars: :hex_upper))
iex> Token.generate()
"6E908C2A1AA7BF101E7041338D43B87266AFA73734F423B6C3C3A17599F40F2A"
```

Note this is much more intuitive than guess, or simply not knowing, how much entropy your random IDs actually have.


### General Note

The mathematical approximations used by **Puid** always favor conservative estimatation:

- overestimate the **bits** needed for a specified **total** and **risk**
- overestimate the **risk** of generating a **total** number of **puid**s
- underestimate the **total** number of **puid**s that can be generated at a specified **risk**



[TOC](#TOC)

### <a name="Installation"></a>Installation

Add `puid` to `mix.exs` dependencies:

```elixir
def deps,
  do: [
    {:puid, "~> 2.1"}
  ]
```

Update dependencies

```bash
mix deps.get
```

### <a name="ModuleAPI"></a>Module API

`Puid` modules have the following functions:

- **generate/0**: Generate a random **puid**
- **total/1**: total **puid**s which can be generated at a specified `risk`
- **risk/1**: risk of generating `total` **puid**s
- **encode/1**: Encode `bytes` into a **puid**
- **decode/1**: Decode a `puid` into **bytes**
- **info/0**: Module information

The `total/1`, `risk/1` functions provide approximations to the **risk** of a repeat in some **total** number of generated **puid**s. The mathematical approximations used purposely _overestimate_ **risk** and _underestimate_ **total**.

The `encode/1`, `decode/1` functions convert `String.t()` **puid**s to and from `bitstring` **bits** to facilitate binary data storage, e.g. as an **Ecto** type.

The `info/0` function returns a `Puid.Info` structure consisting of:

- source characters
- name of pre-defined `Puid.Chars` or `:custom`
- entropy bits per character
- total entropy bits
  - may be larger than the specified `bits` since it is a multiple of the entropy bits per
    character
- entropy representation efficiency
  - ratio of the **puid** entropy to the bits required for **puid** string representation
- entropy source function
- **puid** string length

#### Example

```elixir
iex> defmodule(SafeId, do: use(Puid))

iex> SafeId.generate()
"CSWEPL3AiethdYFlCbSaVC"

iex> SafeId.total(1_000_000)
104350568690606000

iex> SafeId.risk(1.0e12)
9007199254740992

iex> SafeId.decode("CSWEPL3AiethdYFlCbSaVC")
<<9, 37, 132, 60, 189, 192, 137, 235, 97, 117, 129, 101, 9, 180, 154, 84, 32>>

iex> SafeId.encode(<<9, 37, 132, 60, 189, 192, 137, 235, 97, 117, 129, 101, 9, 180, 154, 84, 2::size(4)>>)
"CSWEPL3AiethdYFlCbSaVC"

iex> SafeId.info()
%Puid.Info{
  characters: "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-_",
  char_set: :safe64,
  entropy_bits: 132.0,
  entropy_bits_per_char: 6.0,
  ere: 0.75,
  length: 22,
  rand_bytes: &:crypto.strong_rand_bytes/1
}
```

### <a name="Characters"></a>Characters

There are 19 pre-defined character sets:

| Name              | Characters                                                                                    |
| :---------------- | :-------------------------------------------------------------------------------------------- |
| :alpha            | ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz                                          |
| :alpha_lower      | abcdefghijklmnopqrstuvwxyz                                                                    |
| :alpha_upper      | ABCDEFGHIJKLMNOPQRSTUVWXYZ                                                                    |
| :alphanum         | ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789                                |
| :alphanum_lower   | abcdefghijklmnopqrstuvwxyz0123456789                                                          |
| :alphanum_upper   | ABCDEFGHIJKLMNOPQRSTUVWXYZ0123456789                                                          |
| :base16           | 0123456789ABCDEF                                                                              |
| :base32           | ABCDEFGHIJKLMNOPQRSTUVWXYZ234567                                                              |
| :base32_hex       | 0123456789abcdefghijklmnopqrstuv                                                              |
| :base32_hex_upper | 0123456789ABCDEFGHIJKLMNOPQRSTUV                                                              |
| :crockford32      | 0123456789ABCDEFGHJKMNPQRSTVWXYZ                                                              |
| :decimal          | 0123456789                                                                                    |
| :hex              | 0123456789abcdef                                                                              |
| :hex_upper        | 0123456789ABCDEF                                                                              |
| :safe_ascii       | !#$%&()\*+,-./0123456789:;<=>?@ABCDEFGHIJKLMNOPQRSTUVWXYZ[]^\_abcdefghijklmnopqrstuvwxyz{\|}~ |
| :safe32           | 2346789bdfghjmnpqrtBDFGHJLMNPQRT                                                              |
| :safe64           | ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789-\_                             |
| :symbol           | !#$%&()\*+,-./:;<=>?@[]^\_{\|}~                                                               |
| :wordSafe32       | 23456789CFGHJMPQRVWXcfghjmpqrvwx                                                              |

Any `String` of up to 256 unique characters can be used for **`puid`** generation, with custom characters optimized in the same manner as the pre-defined character sets. The characters must be unique. This isn't strictly a technical requirement, **PUID** could handle duplicate characters, but the resulting randomness of the IDs is maximal when the characters are unique, so **PUID** enforces that restriction.

#### Description of non-obvious character sets

| Name              | Description                                                |
| :---------------- | :--------------------------------------------------------- |
| :base16           | https://datatracker.ietf.org/doc/html/rfc4648#section-8    |
| :base32           | https://datatracker.ietf.org/doc/html/rfc4648#section-6    |
| :base32_hex       | Lowercase of :base32_hex_upper                             |
| :base32_hex_upper | https://datatracker.ietf.org/doc/html/rfc4648#section-7    |
| :crockford32      | https://www.crockford.com/base32.html                      |
| :safe_ascii       | Printable ascii that does not require escape in String     |
| :safe32           | Alpha and numbers picked to reduce chance of English words |
| :safe64           | https://datatracker.ietf.org/doc/html/rfc4648#section-5    |
| :wordSafe32       | Alpha and numbers picked to reduce chance of English words |

Note: :safe32 and :wordSafe32 are two different strategies for the same goal.

[TOC](#TOC)

## <a name="Comparisons"></a>Comparisons

As described in the [overview](https://github.com/puid/.github/blob/2381099d7f92bda47c35e8b5ae1085119f2a919c/profile/README.md), **PUID** aims to be a general, flexible mechanism for creating random string for use as random IDs. The following comparisons to other Elixir random ID generators is with respect to the issues of random ID generation described in that overview.

[TOC](#TOC)

### <a name="Common_Solution"></a>[Common Solution](https://gist.github.com/dingosky/86328fc8b51d6b3037087ab1a8d14b4f#file-common_id-ex)

#### Comments

- Entropy source: Generating indexes via a PRNG is straightforward, though wasteful when compared to bit slicing. Generating indexes via a CSPRNG is not straightforward except for hex characters.
- Characters: Full control
- Captured entropy: Indirectly specified via ID length

#### Timing

**PUID** is much faster.

```
Generate 100000 random IDs with 128 bits of entropy using alphanumeric characters

  Common Solution   (PRNG) : 4.977226
  Puid              (PRNG) : 0.831748

  Common Solution (CSPRNG) : 8.435073
  Puid            (CSPRNG) : 0.958437
```

[TOC](#TOC)

### <a name="misc_random"></a>[misc_random](https://github.com/gutschilla/elixir-helper-random)

#### Comments

- Entropy source: No control. Fixed to PRNG `:random.uniform/1`
- Characters: No control. Fixed to `:alphanum`
- Captured entropy: Indirectly specified via ID length

#### Timing

Quite slow compared to **PUID**

```code
Generate 50000 random IDs with 128 bits of entropy using alphanum characters

  Misc.Random (PRNG) : 12.196646
  Puid        (PRNG) : 0.295741

  Misc.Random (CSPRNG) : 11.9858
  Puid        (CSPRNG) : 0.310417
```

[TOC](#TOC)

### <a name="nanoid"></a>[nanoid](https://github.com/railsmechanic/nanoid)

#### Comments:

- Entropy source: Limited control; choice of CSPRNG or PRNG
- Characters: Full control
- Captured entropy: Indirectly specified via ID length

#### Timing:

**nanoid** is much slower than **PUID**

```
  Generate 75000 random IDs with 126 bits of entropy using safe64 characters

    Nanoid (CSPRNG) : 6.354221
    Puid   (CSPRNG) : 0.226448

    Nanoid (PRNG) : 1.229842
    Puid   (PRNG) : 0.31025

  Generate 75000 random IDs with 195 bits of entropy using alphanum characters

    Nanoid (CSPRNG) : 10.295134
    Puid   (CSPRNG) : 0.809756

    Nanoid (PRNG) : 1.678025
    Puid   (PRNG) : 0.808203
```

[TOC](#TOC)

### <a name="Randomizer"></a>[Randomizer](https://github.com/jeremytregunna/randomizer)

#### Comments

- Entropy source: No control
- Characters: Limited to five pre-defined character sets
- Captured entropy: Indirectly specified via ID length

#### Timing

Slower than **PUID**

```
Generate 100000 random IDs with 128 bits of entropy using alphanum characters

  Randomizer   (PRNG) : 1.201281
  Puid         (PRNG) : 0.829199

  Randomizer (CSPRNG) : 4.329881
  Puid       (CSPRNG) : 0.807226
```

[TOC](#TOC)

### <a name="SecureRandom"></a>[SecureRandom](https://github.com/patricksrobertson/secure_random.ex)

#### Comments

- Entropy source: No control. Fixed to `:crypto.strong_rand_bytes/1`
- Characters: Limited control for 3 specified use cases
- Captured entropy: Indirectly specified via ID length

#### Timing

About the same as **PUID** when using CSPRNG

```
Generate 500000 random IDs with 128 bits of entropy using hex characters

  SecureRandom (CSPRNG) : 1.19713
  Puid         (CSPRNG) : 1.187726

Generate 500000 random IDs with 128 bits of entropy using safe64 characters

  SecureRandom (CSPRNG) : 2.103798
  Puid         (CSPRNG) : 1.806514
```

[TOC](#TOC)

### <a name="ulid"></a>[ulid](https://github.com/ulid/spec)

#### Comments

- Entropy source: No control. Fixed to CSPRNG (per spec)
- Characters: No control. Fixed to :base32
- Captured entropy: 80-bits per timestamp context

A significant characteristic of **ulid** is the generation of lexicographically sortable IDs. This is not a goal for **PUID**; however, one could use **PUID** to generate such IDs by prefixing a timestamp to a generated **puid**. Such a solution would be similar to **ulid** while still providing full control to **entropy source**, **characters**, and **captured entropy** per timestamp context.

#### Timing

**ulid** and **PUID** are not directly comparable with regard to speed.

[TOC](#TOC)

### <a name="UUID"></a>[UUID](https://github.com/zyro/elixir-uuid)

#### Comments

- Entropy source: No control. Fixed to `crypto.strong_rand_bytes/1`
- Character: No control. Furthermore, string representation is inefficient
- Capture entropy: No control. Fixed to 122 bits

#### Timing

Similar to **PUID** when using CSPRNG

```code
Generate 500000 random IDs with 122 bits of entropy using hex
  UUID     : 1.925131
  Puid hex : 1.823116

Generate 500000 random IDs with 122 bits of entropy using safe64
  UUID        : 1.751625
  Puid safe64 : 1.367201
```

[TOC](#TOC)

# Probably Unique IDentifier
PUID aims to be a general, flexible mechanism for creating random string for use as random IDs. Library implementations also strive to be fast and efficient.

For the purposes of PUID, random IDs are a bunch of random strings generated for use as IDs within a specified context. (See Understanding Random IDs for detailed overview.) As such, random ID generation can be viewed as a transformation of some entropy source into a string representation of captured entropy. A general purpose library for such transformation should provide flexible control over the entropy source, the characters used, and the resulting captured randomness.

# ID Generation

#### Entropy source
PUID provide full control over the bytes used as input into the transformation process, while also providing three default sources: CSPRNG, PRNG and Fixed Bytes.

* CSPRNG: Crytographically Secure Pseudo Random Number Generator. This entropy source is recommended for IDs used in context where security is of importance. Each library utilizes language specific mechanisms to access the crytographically secure random bytes provided by the underlying system.

* PRNG: Pseudo Random Number Generator. This entropy source also provides excellent randomess but may be susceptible to certain security related attacks. Again, each library utilizes language specific mechanisms to access randomness provided by the underlying system.

* Fixed Bytes: This entropy source is not random, but provides deterministic bytes useful for testing purposes. An application can use CSPRNG, PRNG, or a custom entropy source for production and configure the use of Fixed Bytes for deterministic testing.

#### Characters
PUID provides full control over the characters used in generated IDs. A number of predefined character sets are provided, and custom characters (including unicode) can also be specified. To maximize captured entropy, PUID enforces the characters used must be unique.

#### Captured randomness
The randomness of IDs is actually the ultimate metric in random ID generation. Curiously, many random ID generators don't even specify what that metric is or don't provide any direct way to specify its value. Given the importance of the metric, it should be explicitly clear how random random IDs are. PUID provides direct specification of the bits of entropy desired for IDs, as well as an intuitive way to specify desired randomness in a total number of generated IDs given a specific risk.

#### Correctness
Each PUID library includes extensive test suites for correctness:

Fixed Bytes entropy sources provide bit level inspection of the correctness in transforming source entropy into random IDs.

Character distribution across large numbers of IDs is tested for uniform distribution using chi-square analysis. These tests are independently and directly performed with the Elixir and Python libraries. All libraries validate ID generation by using comparing output of chi-square tested data with expected output.

#### Efficiency
The general, flexible use of PUID is its primary goals. However, careful consideration has been taken to make the library implementations fast and efficient as well:

PUID produces IDs from its entropy source using a low-level bit slicing scheme that only uses the number of bits necessary to (conceptually) index into an array of input characters. For example, when using 26 lowercase alpha characters, such an index would require at most 5 bits, so PUID sliced input entropy bytes 5 bits at a time. This is in direct contrast to using, for example, a random integer produced from an IEEE 754 float that may well have 53 actual bits of entropy.

***The PUID bit-slicing process maintains unused bits for use in generating the next puid.***

PUID uses the absolute minimum number of bits to produce an ID. Consider, for example, using the 10 vowel characters aeiouAEIOU for IDs. The bit-slicing mechanism used by PUID slices 4 bits at a time to produce a (conceptual) index to determine the next character. One bit-slicing strategy would be to inspect the index and, if greater than 9, throw the 4 bits away and repeat with the next 4 bit slice. Although this would work, it actually wastes bits, so PUID takes the process one step further. As example, suppose the sliced bits were 11xx, that is, the first two sliced bits are 11. Regardless of the other 2 bits, the index from all 4 bits will be greater than 9; therefore PUID only throws away the first 2 bits and uses the xx bits as the beginning of the next bit slice. In this example, the "bit saving" scheme saves 32% of the bits that would be tossed.

***PUID libraries do not actually index into an array of characters, but rather encodes bit-sliced indexes directly for faster ID generation.***


