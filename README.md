# Quimby

Quimby makes it easy to add subcommands to a Go executable, with their own
subcommands and automatic help output. It's a wrapper around the "flag"
package.

## Example usage

Declare a struct type that embeds `*quimby.Flagger`, along with an fields you
want to capture as flags.

```Go
type Echo struct {
	*quimby.Flagger
        echoed string
}
```

Now we need to make our type implement the `quimby.Command` interface. That
requires three methods that aren't already provided by `*quimby.Flagger`:

```Go

func (c *Echo) Desc() string {
	return "Echo the input string."
}

func (c *Echo) SetFlags() {
	c.StringVar(&c.echoed, "input", "", "Text that will be echoed")
}

func (c *Echo) Run() {
	fmt.Println(c.echoed)
}
```

Maybe we write another dozen commands after that. Maybe we implement `ls`. When
we want to actually wire them up, we do something like this:

```
package main

import "github.com/redneckbeard/quimby"

func main() {
	quimby.Add(&Echo{}, &Ls{})
	quimby.Run()
}
```

Now we've got a program to compile. Assuming this is somewhere on your `GOPATH`
as `nix`, we can install it and run it. With no arguments, it will list our
commands.

```bash
$ go install nix
$ nix
Available commands:
  help
  echo
  ls
Type 'nix help <command>' for more information on a specific command.
```

The `help` command is built in, because I care about you. Running it on our
`echo` command will display the return value of the `Desc()` method we
implemented, plus output info about all the flags we registered.

```bash
$ nix help echo
Echo the input string.

Options:
-input     Text that will be echoed
```

Running the `echo` command itself also works.

```bash
$ nix echo -input="Here's some text."
Here's some text.
```
