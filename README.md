[![GoDoc](https://godoc.org/github.com/bitfield/qrand?status.png)](http://godoc.org/github.com/bitfield/qrand)[![Go Report Card](https://goreportcard.com/badge/github.com/bitfield/qrand)](https://goreportcard.com/report/github.com/bitfield/qrand)[![CircleCI](https://circleci.com/gh/bitfield/qrand.svg?style=svg)](https://circleci.com/gh/bitfield/qrand)

```go
import "github.com/bitfield/qrand"
```

# What is `qrand`?

`qrand` is a Go library that provides random numbers generated by a non-deterministic, quantum-mechanical process.

# Sources of randomness

 Most computer random number generators (RNGs) use a deterministic process, which means that given an initial seed value, the sequence of generated numbers is predictable.

For example, Go's standard `math/rand` library uses a fairly simple algorithm to generate a random-looking, but still deterministic sequence of numbers. For most applications this is absolutely fine when seeded with a suitable value, such as the current Unix time in nanoseconds. For any cryptographic purposes, though, `math/rand` is entirely unsuitable, and we should use `crypto/rand` instead.

`crypto/rand` will use the most secure randomness source provided by the operating system; for example, on Linux systems this might be the `/dev/urandom` device. While this is still a pseudo-random source, it uses environmental 'noise' such as I/O activity, keystrokes, and so on, to generate numbers which are in practice (though not in principle) unpredictable.

For very high-security applications, though, we can use quantum-mechanical sources, such as the cosmic microwave background radiation ([Lee and Cleaver 2017](https://www.sciencedirect.com/science/article/pii/S2405844017310897)). The outcomes of quantum measurements, such as the spin of an electron or the polarization of a photon, are _in principle_ unpredictable, to the best of our knowledge ([Bierhorst et al. 2018](https://www.nature.com/articles/s41586-018-0019-0)).

# The ANU randomness server

Various types of hardware quantum RNG devices are available. Australia National University provides a [public quantum randomness source](http://qrng.anu.edu.au/index.php) generated by a device that uses a laser to measure the quantum fluctuations of the vacuum ([Symul, Assad, and Lam 2011](https://aip.scitation.org/doi/10.1063/1.3597793); [Haw et al. 2015](https://journals.aps.org/prapplied/abstract/10.1103/PhysRevApplied.3.054004)).

The ANU website provides several fun and interesting ways to look at the random data. It also provides an API so that we can make use of it in programs. The `qrand` library is a Go client for this API that mirrors the standard library's `crypto/rand`, so that it can be used anywhere `crypto/rand` is applicable.

# Why do I need quantum randomness?

You don't. The standard randomness source provided by your operating system, available via `crypto/rand`, is almost certainly good enough for any application requiring strong randomness, such as cryptography (otherwise, we're all in trouble).

However, it's fun to use a source of randomness which is _entirely_ non-deterministic (so far as we know) and provided directly by the Universe itself, via the quantum state of the vacuum. `qrand` is used, for example, to generate random hexagrams from the I Ching by the [`yijing`](https://github.com/bitfield/yijing) library.

If you have an interesting application for `qrand`, [let me know!](mailto:john@bitfieldconsulting.com)

# Using `qrand` in programs

Let's look at the [example program](example/main.go) to see how to generate a sequence of random bytes:

```go
package main

import (
	"github.com/bitfield/qrand"
	"fmt"
	"log"
)

func main() {
	numbers := make([]byte, 10)
	_, err := qrand.Read(numbers)
	if err != nil {
		log.Fatal(err)
	}
	fmt.Println(numbers)
}
```

As you can see, this is very similar to the [equivalent `crypto/rand` program](https://golang.org/pkg/crypto/rand/#example_Read). The only difference is using `qrand.Read` instead of `rand.Read`.

# Convenience wrappers for `qrand`

The `crypto/rand` API is rather basic, so it can be convenient to use a wrapper such as that described by Nelz Carpentier in [A Tale of Two `rand`s](https://blog.gopheracademy.com/advent-2017/a-tale-of-two-rands/). This lets us use nice `math/rand` functions such as `rand.Intn()`, but sourcing data from `qrand`.