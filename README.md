> [!WARNING]
> Technical fork, no longer actively maintained

# `blstrs`

> Implementation of BLS12-381 pairing-friendly elliptic curve construction, using the [blst](https://github.com/supranational/blst) library as backend.

## Pronounciation

Most important section, the name is pronounced `blasters`.

## Supported Platforms

Due to the assembly based nature of the implementation in `blst`, currently only the following architectures are supported

- `x86_64`,
- `aarch64`.

## BLST Portability

To enable portable features when building the blst dependency, use the 'portable' feature: `--features portable`.


## Benchmarking

```
$ cargo bench --features __private_bench
```


## BLS12 Parameterization

BLS12 curves are parameterized by a value *x* such that the base field modulus *q* and subgroup *r* can be computed by:

* q = (x - 1)<sup>2</sup> ((x<sup>4</sup> - x<sup>2</sup> + 1) / 3) + x
* r = (x<sup>4</sup> - x<sup>2</sup> + 1)

Given primes *q* and *r* parameterized as above, we can easily construct an elliptic curve over the prime field F<sub>*q*</sub> which contains a subgroup of order *r* such that *r* | (*q*<sup>12</sup> - 1), giving it an embedding degree of 12. Instantiating its sextic twist over an extension field F<sub>q<sup>2</sup></sub> gives rise to an efficient bilinear pairing function between elements of the order *r* subgroups of either curves, into an order *r* multiplicative subgroup of F<sub>q<sup>12</sup></sub>.

In zk-SNARK schemes, we require F<sub>r</sub> with large 2<sup>n</sup> roots of unity for performing efficient fast-fourier transforms. As such, guaranteeing that large 2<sup>n</sup> | (r - 1), or equivalently that *x* has a large 2<sup>n</sup> factor, gives rise to BLS12 curves suitable for zk-SNARKs.

Due to recent research, it is estimated by many that *q* should be approximately 384 bits to target 128-bit security. Conveniently, *r* is approximately 256 bits when *q* is approximately 384 bits, making BLS12 curves ideal for 128-bit security. It also makes them ideal for many zk-SNARK applications, as the scalar field can be used for keying material such as embedded curve constructions.

Many curves match our descriptions, but we require some extra properties for efficiency purposes:

* *q* should be smaller than 2<sup>383</sup>, and *r* should be smaller than 2<sup>255</sup>, so that the most significant bit is unset when using 64-bit or 32-bit limbs. This allows for cheap reductions.
* F<sub>q<sup>12</sup></sub> is typically constructed using towers of extension fields. As a byproduct of [research](https://eprint.iacr.org/2011/465.pdf) for BLS curves of embedding degree 24, we can identify subfamilies of BLS12 curves (for our purposes, where x mod 72 = {16, 64}) that produce efficient extension field towers and twisting isomorphisms.
* We desire *x* of small Hamming weight, to increase the performance of the pairing function.

## BLS12-381 Instantiation

The BLS12-381 construction is instantiated by `x = -0xd201000000010000`, which produces the largest `q` and smallest Hamming weight of `x` that meets the above requirements. This produces:

* q = `0x1a0111ea397fe69a4b1ba7b6434bacd764774b84f38512bf6730d2a0f6b0f6241eabfffeb153ffffb9feffffffffaaab` (381 bits)
* r = `0x73eda753299d7d483339d80809a1d80553bda402fffe5bfeffffffff00000001` (255 bits)

Our extension field tower is constructed as follows:

1. F<sub>q<sup>2</sup></sub> is constructed as F<sub>q</sub>(u) / (u<sup>2</sup> - β) where β = -1.
2. F<sub>q<sup>6</sup></sub> is constructed as F<sub>q<sup>2</sup></sub>(v) / (v<sup>3</sup> - ξ) where ξ = u + 1
3. F<sub>q<sup>12</sup></sub> is constructed as F<sub>q<sup>6</sup></sub>(w) / (w<sup>2</sup> - γ) where γ = v

Now, we instantiate the elliptic curve E(F<sub>q</sub>) : y<sup>2</sup> = x<sup>3</sup> + 4, and the elliptic curve E'(F<sub>q<sup>2</sup></sub>) : y<sup>2</sup> = x<sup>3</sup> + 4(u + 1).

The group G<sub>1</sub> is the *r* order subgroup of E, which has cofactor (x - 1)<sup>2</sup> / 3. The group G<sub>2</sub> is the *r* order subgroup of E', which has cofactor (x<sup>8</sup> - 4x<sup>7</sup> + 5x<sup>6</sup> - 4x<sup>4</sup> + 6x<sup>3</sup> - 4x<sup>2</sup> - 4x + 13) / 9.

### Generators

The generators of G<sub>1</sub> and G<sub>2</sub> are computed by finding the lexicographically smallest valid `x`-coordinate, and its lexicographically smallest `y`-coordinate and scaling it by the cofactor such that the result is not the point at infinity.

#### G1

```
x = 3685416753713387016781088315183077757961620795782546409894578378688607592378376318836054947676345821548104185464507
y = 1339506544944476473020471379941921221584933875938349620426543736416511423956333506472724655353366534992391756441569
```

#### G2

```
x = 3059144344244213709971259814753781636986470325476647558659373206291635324768958432433509563104347017837885763365758*u + 352701069587466618187139116011060144890029952792775240219908644239793785735715026873347600343865175952761926303160
y = 927553665492332455747201965776037880757740193453592970025027978793976877002675564980949289727957565575433344219582*u + 1985150602287291935568054521177171638300868978215655730859378665066344726373823718423869104263333984641494340347905
```

## License

<sup>
Licensed under either of <a href="LICENSE-APACHE">Apache License, Version
2.0</a> or <a href="LICENSE-MIT">MIT license</a> at your option.
</sup>

<br/>

<sub>
Unless you explicitly state otherwise, any contribution intentionally submitted
for inclusion in this crate by you, as defined in the Apache-2.0 license, shall
be dual licensed as above, without any additional terms or conditions.
</sub>
