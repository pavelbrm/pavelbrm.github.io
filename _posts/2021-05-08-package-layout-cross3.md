---
layout: post
title: "The Style Guide: Package Layout 6 – Cross-Platform Packages Part III"
category: Golocron
tags: [golang, m1u2]
---

> When you develop your opinions on the basis of weak evidence, you will have difficulty interpreting subsequent information that contradicts these opinions, even if this new information is obviously more accurate.
>
> – Nassim Nicholas Taleb

The third part of the cross-platform series focuses on the practical aspect of the question. It provides several examples of code organisation and tests in a cross-plaform package.


<!--more-->

![](/assets/m1u2_6.jpg)

<sub>_Photo credit:_ <a href="https://unsplash.com/@heysupersimi" target="_blank">_Simone Hutsch_</a></sub>


## Before We Begin

By now, we have learned a few foundational principles and tools that guide and help us when working on a piece of cross-platform software. Next step is to look at how to combine and apply them to make software maintainable and the process of maintenance as smooth as possible. The ultimate goal is to make code simple and easy to work with; we also strive to simplify support of such software.

The section introduces practical advice on how to design packages and files better in a cross-platform project. This includes:
- [Keep Platform-Dependent Code at Minimum](#keep-platform-dependent-code-at-minimum)
- [Use Simple Branching in Trivial Cases](#use-simple-branching-in-trivial-cases)
- [No Platform-Specific Code in Main](#no-platform-specific-code-in-main)
- [Use File Suffix by Default](#use-file-suffix-by-default)
- [Use Build Tags in Mixed Cases](#use-build-tags-in-mixed-cases)
- [An Advanced Example](#an-advanced-example)
- [Strive for Platform-Independent Tests](#strive-for-platform-independent-tests)
- [Provide Cross-Platform Tests Only When You Must](#provide-cross-platform-tests-only-when-you-must)


We begin with a simple and straightforward use case and will gradually strengthen requirements. The code snippets shown in this draft can be copied and executed.


## Keep Platform-Dependent Code at Minimum

Keep the amount of platform-specific code at a possible minimum.

Sounding very simple, it's one of the hardest to follow, and the most often ignored piece of advice. By its nature, it goes hand-in-hand with the common sense in package design – keep the public API of a package as narrow as possible.

The best way to keep things simple is to have no cross-platform code. But often that's not possible, and some code has to be made platform-dependent. In such case, you have to keep the amount of such code as small as possible.

Having learned the tools for conditional compilation, you know that it's possible to conditionally include code in the final binary depending on the constraints specified at the build time. This leads to a question about what code to pull into files that are conditionally included? Or, in a more general form, what parts of the code are better to be made cross-platform?

Looking at the problem from a general software design perspective, our building blocks in code are:
- types
- methods
- and helper functions.

Therefore, when working in a cross-platform context, these are what can have varying implementations between platforms supported by a project. However, the guideline says that it's good to keep the code at a minimum. This helps in deriving guiding principles for writing cross-platform code.

In general, when working with varying implementations for the same functionality for several platforms, bear in mind the following suggestions:
- in the worst case, types are implemented differently for each platform, but obey the same interface/contract
- a slightly better option is to allow methods vary between platforms
- the best option is to pull platform specifics into helper functions.

The insightful reader might have already noticed that these guidelines do support and reflect two of the basic principles introduced at the beginning of this unit, namely:
- the business logic must be platform-agnostic
- and keep the amount of cross-platform code at a possible minimum.

The first of the two is supported by the fact that by default you aim to pull platform-specifics in to helpers. This guarantees that methods and types do not depend on a platform, thus less business logic is leaking to the platform.

The second is supported by the fact that aiming for having no details leaked to type and method levels, a lower amount of platform-dependent code is automatically guaranteed.

One warning should be made though. The guideline does not prevent you from introducing platform-specific **helper types** when it helps to improve the logic of software and its design. What should be avoided, however, is platform-specific public API of a package. If a public method or type depends on a platform, then decouple the public part and use cross-platform private implementations that are not exposed to the user. This drastically improves both, the user and developer experience. Being free to change the implementation without the risk of breaking user's applications is vital for productive and efficient maintenance of any software, but even more so in the cross-platform world.


## Use Simple Branching in Trivial Cases

Use simple branching in trivial cases.

A simplest form of cross-platform code is setting a variable within an `if` or `switch` statement depending on values of `runtime.GOOS`, `runtime.GOARCH` or `runtime.Version()`.

Such code does not require any build constraints (reminder, build constraints can be expressed as a suffix in the name of a file, or as a build tag). The result depends on the values of the corresponding constants or the returned value of the `Version()` function, which in turn depend on the system where the program is running and/or version of Go used during the build process.

However, it's tempting to abuse this method. Beware that it is only helpful in very basic cases like choosing the appropriate extension of a file name or something similar. Never implement different **behaviour** based on branching depending on the aforementioned values. This leads to fragile architecture and complicates testing, debugging and maintenance overall.

This method helps to choose what path the execution will follow **at runtime**, but it *does not imply* conditional compilation. If implementations vary between platforms, and are not available on them at the same time, then builds will be broken.

Essentially, one of the most reasonable uses of this option is to determine and/or report what actual platform the code is _running_ on.

Don't confuse runtime conditions with compile-time conditions.


## No Platform-Specific Code in Main

Keep the `main` function, as well as the `main` package, free from platform-dependant code.

This guideline is a natural and logical consequence of the fact that the `main` package must contain the smallest possible amount of code. When initialisation steps are encapsulated in a separate single or multiple packages, it's possible to have platform-specific initialisation wherever it is required. Other advantages of this are improved testability and separation of concerns.

Technically, the rules that manage conditional compilation apply to the `main` package and the files in it, as to any other package. However, if your `main` contains platform-specific code, it suggests there is _likely_ a problem with design and architecture – implementation details have leaked at the highest level of the application hierarchy. The `main` package must be as free as possible from any details.


## Use File Suffix by Default

Use the file suffix to manage cross-platform code by default, i.e. when the conditions for compilation are simple.

What this suggestion means is that in simple cases where the platforms you need to support are discrete and don't share commonalities, the simplest possible way is to use suffixes with files containing code for a particular platform, be it different operating systems, architectures, or a combination of the two criteria.

For instance, suppose you have a CLI application that must support Linux and Windows, and the architecture does not matter (as in `amd64` or `386`), and something requires a different implementation for each of the platforms. In this case, we have a simplest condition possible – two different operating systems. The best way to express it is to use suffixes:

```bash
└── something
    ├── something.go
    ├── something_linux.go
    └── something_windows.go
```

Nothing else is required. Not even a single tag. Then, as discussed earlier, there are two ways to build it:
- on the same machine, using cross compilation:

```bash
# For Linux.
GOOS=linux go build -o bin/myapp-linux ./cmd/myapp

# For Windows.
GOOS=windows go build -o bin/myapp-windows.exe ./cmd/myapp
```

- on machines with the corresponding platforms the environment variable is not needed:

```bash
# On Linux.
go build -o bin/myapp-linux ./cmd/myapp

# On Windows.
go build -o bin/myapp-windows.exe ./cmd/myapp
```


## Use Build Tags in Mixed Cases

Use build tags and suffixes in mixed cases, i.e. when some platforms have their unique implementations, but some share code.

Let's extend the previous example by the following requirement: the app should also support macOS, `darwin` in Go's vocabulary, and the implementation of the platform-specific part is exactly the same as it is for Linux. This is a pretty common situation for macOS. Some parts work close or similar to the `linux` platform, some are similar to those in `freebsd`, while some are unique to `darwin` itself.

In the case of our example, we have two options:
- implement the new requirement separately for `darwin`, hence in its own file, `something_darwin.go`
- or re-use the existing implementation which, for the sake of exercise, is assumed to be the same.

The former option has a rather weak advantage of not bothering with conditional compilation, and just relies on the automatic choice the compiler makes. A big disadvantage though is unjustified duplication, which clearly can and should be avoided. And this is exactly what the second option is about.

In order to re-use the existing implementation for `darwin`, and continue supporting `linux`, we need to make a few changes:
- first off, stop relying on the file suffix, as there is no file suffix that would include both `linux` and `darwin`, and exclude all other platforms at the same time. This means, we shall rename the file
- secondly, we need to specify a proper build tag to tell the compiler when it should include the file.

How to choose a new name for the `something_linux.go` file? We can't simply append `_darwin` at the end, as this file becomes suffixed with `darwin`, and will be only included for this platform. The name should not be a valid word from the list of operating systems, platforms and any other words supported by Go. Here are a few examples:
- `something_darlin.go`
- `something_lindar.go`
- `something_linuxdarwin.go`.

However, this particular case is very often. What's common between Linux, macOS, and many other Unix and Unix-like operating systems? Right, they are, though to varying degrees, compliant with POSIX. So a pretty common thing to see in the standard library and other cross-platform projects is the use of the word `posix` as a suffix for cases when Linux and other operating systems from the Unix family share same code. Going further, you can also often run into cases when you have something common among all or many Unices, but different for Linux. In such case, for example, code that is the same for FreeBSD, OpenBSD, NetBSD and macOS can be put into a file with a name suffixed by `_unix.go`. Keep this in mind, and don't reinvent the wheel.

Alright, the name problem is solved. The existing file needs to be renamed from `something_linux.go` to `something_posix.go`. The name does not tell anymore the compiler when to include and exclude the file, but a tag will do. The code in the file should be built for Linux or macOS. That `or` is really important, as it's a condition. It does not make sense to express it as `and`, since this is an impossible combination (an operating system cannot be Linux and macOS at the same time). Here is what the resulting tag should look like:

```golang
// +build linux darwin
```

Also, keep in mind the rules about the placement of a build tag.

This is what our example was like before the changes:
- directory listing:

```bash
.
├── bin
├── cmd
│   └── myapp
│       └── main.go
└── something
    ├── something.go
    ├── something_linux.go
    └── something_windows.go
```

- and the content of the `something_linux.go` file:

```golang
package something

const (
	Feature2 = "this is a version supported by a number of unix-like systems"
)
```

This is what it looks like after the changes:
- directory listing:

```bash
.
├── bin
├── cmd
│   └── myapp
│       └── main.go
└── something
    ├── something.go
    ├── something_posix.go
    └── something_windows.go
```

- and the content of the `something_posix.go` file:

```golang
// +build linux darwin

package something

const (
	Feature2 = "this is a version supported by a number of unix-like systems"
)
```

- building:

```bash
# On/For Mac.
$ GOOS=darwin go build -o bin/myapp-darwin ./cmd/myapp

# Running.
$ ./bin/myapp-darwin
common feature for all platforms
this is a version supported by a number of unix-like systems
```


## An Advanced Example

Here is another question: What if we need to support a third feature which is unique to the Mac, and is unavailable for all other platforms? In this case, we should use the full power available at our disposal, and also avoid exposing the difference to `main`. How can this be accomplished?

We can use our example to illustrate the approach. Let's assume there should be a feature that does something on `darwin` but does not exist/has no effect on other platforms. There are two approaches available:
- mock the behaviour on the platforms that don't have the feature, so it can be called in a platform-independent way, i.e. make it no-op
- or encapsulate the platform-dependent call such that it's not exposed to the platform-independent code, hence is transparent.

The latter is available and really helpful in real life when implementing unique parts nested within a higher-level code that is called by code that is unaware of different platforms. For our example we'll follow the former way, but the second is preferred whenever it's possible.

Also note, that a simple `if runtime.GOOS == "darwin" { // unique code }` won't do the trick. Why? Because it will be attempted to be included for all platforms during compilation, and conditioned only at runtime.

Let's introduce a third feature which should work only on the Mac, while all existing conditions must remain. In order to make this happen, we:
- implement the new feature for `darwin`
- mock it for `windows` and `linix`
- the second step implies that we have to add a Linux-specific file. We'll use the suffix for that.

We also don't want to make a special case for the existing code shared between Linux and macOS. The fact that we're now going to support all three platforms, does not mean we should have a separate platform-specific implementation, just for the sake of separation. As one of the guiding principles says, the more code can be shared and/or abstracted, the better. And if the second feature introduced earlier remains the same for the two platforms, there is absolutely no point in splitting it.

Let's have a look at the project after the necessary have been made.
- directory listing:

```bash
.
├── bin
├── cmd
│   └── myapp
│       └── main.go
└── something
    ├── something.go
    ├── something_darwin.go
    ├── something_linux.go
    ├── something_posix.go
    └── something_windows.go
```

Notice the two new files, `something_darwin.go` and `something_linux.go`.

- the `main.go` file:

```golang
package main

import (
	"fmt"

	"../../something"
)

func main() {
	fmt.Println(something.Feature1)
	fmt.Println(something.Feature2)

	something.DoFeature3()
}
```

Notice how the code does not depend on platform here.

- `something.go`

```golang
package something

const (
	Feature1 = "common feature for all platforms"
)
```

This piece of code is common among all supported platforms.

- `something_darwin.go`

```golang
package something

import (
	"fmt"
)

func DoFeature3() {
	fmt.Println("this is a unique feature for the mac")
}
```

This code is compiled for and executed only when running on the Mac.

- `something_linux.go`

```golang
package something

func DoFeature3() {}
```

Here we mock the missing functionality for Linux.

- `something_posix.go`

```golang
// +build linux darwin

package something

const (
	Feature2 = "this is a version supported by a number of unix-like systems"
)
```

This code remains the same, as it works nicely for the two platforms, hence no change is needed. This file is included for both platforms.

- `something_windows.go`

```golang
package something

const (
	Feature2 = "this is a windows specific version"
)

func DoFeature3() {}
```

Finally, for Windows, the new feature is mocked in this existing file.

Here are a few results of building and running:

```bash
# Build for darwin.
GOOS=darwin go build -o bin/myapp-darwin ./cmd/myapp

# Run on darwin.
$ ./bin/myapp-darwin
common feature for all platforms
this is a version supported by a number of unix-like systems
this is a unique feature for the mac


# Build for linux.
$ GOOS=linux go build -o bin/myapp-linux ./cmd/myapp

# Run on linux.
$ docker run -it --rm -v $(pwd)/bin:/root/bin/ alpine:3.13 ash
% /root/bin/myapp-linux
common feature for all platforms
this is a version supported by a number of unix-like systems
```

As you see, the feature works as expected on macOS, and is non-existent on Linux. _Quod Erat Demonstrandum._


## Strive for Platform-Independent Tests

Write platform-independent tests whenever possible.

This suggestion is corollary to and supports the previous two guidelines. Maintenance of multi-platform code is not a trivial business, and has its cost. By making sure that the code is designed to be as much as possible independent of the platform it's run on, and only really low-level parts do differ, we have hopefully reduced the cost. But the code has to be tested, and since tests are also code, the same rules apply.

Platform-independent approach to tests works best in a case where the result of an operation or behaviour must be the same across all platforms despite different low-level implementation details. In such a case, the low-level code does not necessarily need to be covered, as long as tests are run against all target platforms.

To illustrate this, let's slightly update our example. We'll add a feature that has to work uniformly across all platforms, but will have three different implementations. Let's say we need to add a fourth feature which prints `"this feature has different implementations but yields the same result"` on all three supported operating systems (here, as in `GOOS`).

This is what the code looks like:
- this is our platform-independent high-level code, `something.go`:

```golang
package something

import (
	"fmt"
)

const (
	Feature1 = "common feature for all platforms"
)

func RunFeature4() {
	fmt.Println(Feature4())
}

func Feature4() string {
	return doFeature4()
}
```

Notice the `RunFeature4` function which internally calls the `doFeature4` function. Where is it declared? It's not declared in any file that is visible for all platforms simultaneously. Instead, each of the platforms has its own implementation. Here we go:

- for `darwin`, `something_darwin.go`:

```golang
package something

import (
	"fmt"
)

var (
	feature4 = []byte(`this feature has different implementations but yields the same result`)
)

func DoFeature3() {
	fmt.Println("this is a unique feature for the mac")
}

func doFeature4() string {
	return string(feature4)
}
```

Here, the special implementation of `doFeature4` uses the slice of `byte`s as the source.

- for `linux`, `something_linux.go`:

```golang
package something

func DoFeature3() {}

func doFeature4() string {
	return "this feature has different implementations but yields the same result"
}
```

In the world of Linux, things are often more straightforward. We just return a `string`.

- here is `windows`, `something_windows.go`:

```golang
package something

const (
	Feature2 = "this is a windows specific version"
)

var (
	feature4 = []rune(`this feature has different implementations but yields the same result`)
)

func DoFeature3() {}

func doFeature4() string {
	return string(feature4)
}
```

In Windows, our implementation is based on a slice of `rune`s.

- finally, this is how the feature is used, `main.go`:

```golang
package main

import (
	"fmt"

	"../../something"
)

func main() {
	fmt.Println(something.Feature1)
	fmt.Println(something.Feature2)

	something.DoFeature3()
	something.RunFeature4()
}
```

- building and running:

```bash
# Darwin.
$ GOOS=darwin go build -o bin/myapp-darwin ./cmd/myapp
$ ./bin/myapp-darwin
common feature for all platforms
this is a version supported by a number of unix-like systems
this is a unique feature for the mac
this feature has different implementations but yields the same result

# Linux.
$ GOOS=linux go build -o bin/myapp-linux ./cmd/myapp
$ docker run -it --rm -v $(pwd)/bin:/root/bin/ alpine:3.13 ash
% /root/bin/myapp-linux
common feature for all platforms
this is a version supported by a number of unix-like systems
this feature has different implementations but yields the same result
```

The output suggests that the feature works as expected. But how can we prove it remains working over time, especially given that changes to software are made often, not to mention that platforms do not stand still too, and are being constantly updated? Of course, with tests. In this case, the outcome is and must be exactly the same for all platforms. So this outcome **is the result** we want to make sure is always correct, not the implementation detail. If something happens to the implementation, then the broken test will highlight it.

In cases like this, **write platform-independent** tests.

To illustrate this situation, here are two example tests:
- the first checks the feature itself
- the second tests how it works end-to-end.

Here is the `something_test.go` file, which is in effect on all platforms:

```golang
package something_test

import (
	"bufio"
	"os"
	"strings"
	"testing"

	"../something"
)

func TestFeature4(t *testing.T) {
	exp := "this feature has different implementations but yields the same result"
	act := something.Feature4()

	if act != exp {
		t.Errorf("expected: %s, got: %s\n", exp, act)
	}
}

func TestRunFeature4(t *testing.T) {
	stdout := os.Stdout
	r, w, err := os.Pipe()
	if err != nil {
		t.Errorf("failed to create reader and writer: %s\n", err)
		t.FailNow()
	}

	os.Stdout = w
	something.RunFeature4()

	b := bufio.NewReader(r)
	out, err := b.ReadString('\n')
	if err != nil {
		t.Errorf("failed to read from reader: %s\n", err)
		t.FailNow()
	}

	if err := w.Close(); err != nil {
		t.Errorf("failed to close writer: %s\n", err)
		t.FailNow()
	}

	os.Stdout = stdout

	exp := "this feature has different implementations but yields the same result"
	if act := strings.TrimSuffix(out, "\n"); act != exp {
		t.Errorf("expected: %q, got: %q\n", exp, act)
	}
}
```

Notice how neither of the two tests has any knowledge about the platform-specific code. In the first test, `TestFeature4`, we check that the feature is implemented correctly. The second test, `TestRunFeature4`, does a full check, including capturing the result from the standard output.

Here are test results:
- on macOS:

```bash
$ go test ./something/... -v
=== RUN   TestFeature4
--- PASS: TestFeature4 (0.00s)
=== RUN   TestRunFeature4
--- PASS: TestRunFeature4 (0.00s)
PASS
ok  	_/Users/user/projects/xplatform/something	0.005s
```

- and on Linux, using the `golang:1.15` image:

```bash
$ docker run -it --rm -v $(pwd):/root/xplatform golang:1.15 bash
% cd /root/xplatform/
% go test ./something/... -v
=== RUN   TestFeature4
--- PASS: TestFeature4 (0.00s)
=== RUN   TestRunFeature4
--- PASS: TestRunFeature4 (0.00s)
PASS
ok  	_/root/xplatform/something 0.003s
```

The output is exactly the same, and tests pass in both cases. And that's what we should strive for.


## Provide Cross-Platform Tests Only When You Must

Provide platform-specific tests only when you must.

If we had lived in a perfect world, platform-agnostic tests would have been enough. Since this is not the case, under some circumstances we have to provide our code with specialised tests. There are two main reasons for having specialised tests:
- the feature being tested is not available on all platforms, hence there is no way to test it uniformly in accordance with the previous guideline
- or the platform-specific implementation detail is crucial, and requires an extra cautious approach.

Note that the latter of the two reasons is rather rare for most cases of casual and business software. A good illustration of this and the preceding guidelines can be found in the `net` package of the Go's standard library. The vast majority of the package's tests operates on platform-independent code, and only a few parts are provided with specialised tests.

When a single or a group of platforms needs a separate set of tests, follow the same rules as with regular cross-platform code:
- in simple cases, use file name suffixes, e.g. `something_darwin_test.go`
- in cases where a number of platforms share the same implementation and tests, use a meaningful suffix and build tags, e.g. `something_unix_test.go` and `// +build darwin freebsd`.

To give an example, let's add a test to the feature that is unique for the `darwin` platform in the demo code shown previously. As a reminder, this is the platform-specific implementation we're going to test, `something/something_darwin.go`:

```golang
package something

import (
	"fmt"
)

var (
	feature4 = []byte(`this feature has different implementations but yields the same result`)
)

func DoFeature3() {
	fmt.Println("this is a unique feature for the mac")
}

func doFeature4() string {
	return string(feature4)
}
```

In particular, we're interested in the `DoFeature3` function, as it's available only on the Mac; on all other platforms it's mocked as `func DoFeature3() {}`. The implementations are different, so there is no way it can be expressed in a clean way. Of course, one can always write `if runtime.GOOS() == "darwin" { ... }`, but that's rather naïve. Why would we allow such a leak of details to a higher level of abstraction?

Instead, we create a file to hold the tests for the platform of interest, `something_darwin_test.go` in our case, and fill it with test code:

```golang
package something_test

import (
	"bufio"
	"os"
	"strings"
	"testing"

	"../something"
)

func TestDoFeature3(t *testing.T) {
	stdout := os.Stdout
	r, w, err := os.Pipe()
	if err != nil {
		t.Errorf("failed to create reader and writer: %s\n", err)
		t.FailNow()
	}

	os.Stdout = w
	something.DoFeature3()

	b := bufio.NewReader(r)
	out, err := b.ReadString('\n')
	if err != nil {
		t.Errorf("failed to read from reader: %s\n", err)
		t.FailNow()
	}

	if err := w.Close(); err != nil {
		t.Errorf("failed to close writer: %s\n", err)
		t.FailNow()
	}

	os.Stdout = stdout

	exp := "this is a unique feature for the mac"
	if act := strings.TrimSuffix(out, "\n"); act != exp {
		t.Errorf("expected: %q, got: %q\n", exp, act)
	}
}
```

Running the tests emits the following output:
- on macOS

```bash
$ GOOS=darwin go test ./something/... -v
=== RUN   TestDoFeature3
--- PASS: TestDoFeature3 (0.00s)
=== RUN   TestFeature4
--- PASS: TestFeature4 (0.00s)
=== RUN   TestRunFeature4
--- PASS: TestRunFeature4 (0.00s)
PASS
ok  	_/Users/user/projects/xplatform/something	0.006s
```

- and on Linux, the output is still the same as before

```bash
$ docker run -it --rm -v $(pwd):/root/xplatform golang:1.15 bash
% cd /root/xplatform/
% go test ./something/... -v
=== RUN   TestFeature4
--- PASS: TestFeature4 (0.00s)
=== RUN   TestRunFeature4
--- PASS: TestRunFeature4 (0.00s)
PASS
ok  	_/root/xplatform/something 0.003s
```

Organising cross-platform code and tests this way allows for:
- better encapsulation of responsibilities
- better isolation of changes
- more clarity on what, when, where and how
- and this all results into a healthier software and process.

This example shows a couple of important things – that it is possible to have specialised tests if needed, and that the complexity and cost of maintenance grow much faster with each new varying detail. While being able to test the specifics of a platform is important, it's also important to use this ability wisely. The best code is unwritten code; but a better code is the one that checks for the outcome rather than for a specific implementation detail. This supports the guideline – write platform-specific tests only when you must, and try to cover as much as possible at the platform-independent level. It will save you time and energy.

---

To be continued...
