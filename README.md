# Z80 Assembly syntax highlighting

A SublimeText 3 syntax highlighting package for the Z80 Assembly language.

There are many flavours of Z80 assembly and supporting them all is perhaps unrealistic. Therefore the focus of this package is with [Pasmo](http://pasmo.speccy.org/) - a portable Z80 cross assembler - as itself is compatible with numerous other assemblers.

![Screen grab from SublimeText using the Monokai theme](https://github.com/mrcook/Z80Assembly/raw/master/images/sublimetext-z80.png)


### Installation

#### Manual Installation

First, locate the SublimeText [packages folder](http://docs.sublimetext.info/en/latest/basic_concepts.html#the-packages-directory) on your OS, then clone with git:

    $ cd /path/to/sublime/packages/folder
    $ git clone https://github.com/mrcook/Z80Assembly.git


### Assembler compatibility notes

Although I've followed the Pasmo specification (see [pasmo.md](https://github.com/mrcook/Z80Assembly/blob/master/docs/pasmo.md)) this package deviates a little.

- Hexadecimal numbers are expected to be upper case.
- Pasmo identifiers, directives, labels, etc., are expected to be upper case.
- Only double-quoted strings are currently supported.
- Labels starting with an underscore are not supported.

There may be other small deviations - otherwise known as bugs! If you encountered something that is not correct, please consider submitting an issue.


### Roadmap

- Fix the current compatibility issues (as above).
- Support both upper and lower case.
- Support for assemblers beyond just Pasmo.


### Contributing

Pull requests are welcome.


### License

```
Copyright (c) 2018 Michael R. Cook

This work is licensed under the terms of the MIT license.
For a copy, see <https://opensource.org/licenses/MIT>.
```
