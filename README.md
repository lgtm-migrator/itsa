# itsa - Income Tax Self-Assessment

[![Builds](https://github.com/ac000/itsa/actions/workflows/build_tests.yaml/badge.svg)](https://github.com/ac000/itsa/actions/workflows/build_tests.yaml)
[![FreeBSD Build Status](https://api.cirrus-ci.com/github/ac000/itsa.svg)](https://cirrus-ci.com/github/ac000/itsa)
[![Language grade: C/C++](https://img.shields.io/lgtm/grade/cpp/g/ac000/itsa.svg?logo=lgtm&logoWidth=18)](https://lgtm.com/projects/g/ac000/itsa/context:cpp)

itsa is a program for interacting with the UK's HMRC [Make Tax Digital](https://developer.service.hmrc.gov.uk/api-documentation) APIs.

It makes use of [libmtdac](https://github.com/ac000/libmtdac)

# Status

### Credentials

- [2021-03-05] Started the process of obtaining production credentials.
- [2021-11-19] Finally have production credentials! You can see the gory details of the journey [here](https://github.com/ac000/libmtdac/discussions/18).

Unfortunately anyone wishing to use itsa will have to go through the same
procedure by signing up to the
[HMRC Developer HUB](https://developer.service.hmrc.gov.uk/api-documentation)
and registering this application and then applying for production credentials.

Feel free to
[email](mailto:Andrew%20Clayton%20%3candrew%40digital-domain.net%3e?subject=%5bitsa%20registration%5d%20)
me with any questions regarding this particular bit. General questions/issues
should be done through the GitHub [itsa](https://github.com/ac000/itsa)
project.

itsa currently supports the following actions

  - List Self-Employment period obligations
  - Create and update Self-Employment periods
  - Create/Update a Self-Employment annual summary
  - Submit an End-of-Year Statement
  - Submit a final declaration
  - List/view tax calculations
  - View an End-of-Year tax/nics estimate
  - Add/view/amend savings accounts

Currently it gets the required accounting data from a GNUCash SQLite backed
file.

# Building & Installing

itsa is primarily developed under Linux but it also builds and runs under
FreeBSD.

### Linux

itsa has a few dependencies

  - [libmtdac](https://github.com/ac000/libmtdac)
  - [libac](https://github.com/ac000/libac)
  - [libcurl](https://curl.se/libcurl/) (via libmtdac)
  - [jansson](https://digip.org/jansson/)
  - [sqlite](https://www.sqlite.org/index.html)

the last two should already be packaged up for your system.

On Red Hat/Fedora/etc libcurl and jansson can be obtained with

```
$ sudo dnf install libcurl{,-devel} jansson{,-devel} sqlite{,-devel}
```

On Debian (something like...)

```
$ sudo apt-get install libcurl4{,-openssl-dev} libjansson{4,-dev} libsqlite3-{0,dev}
```

Once they are installed you can build the other two libraries.

On Red Hat/Fedora you can build RPMs for these.

First create the rpm-build directory structure

```
$ mkdir -p ~/rpmbuild/{BUILD,RPMS,SOURCES,SPECS,SRPMS}
```

make sure you have the rpm-build package

```
$ sudo dnf install rpm-build
```

```
$ git clone https://github.com/ac000/libmtdac.git
$ cd libmtdac
$ make rpm
$ sudo dnf install ~/rpmbuild/RPMS/x86_64/libmtdac-*
```

```
cd ..
```

```
$ git clone https://github.com/ac000/libac.git
$ cd libac
$ make rpm
$ sudo dnf install ~/rpmbuild/RPMS/x86_64/libac-*
```

```
cd ..
```

and finally itsa itself

```
$ git clone https://github.com/ac000/itsa.git
$ cd itsa
$ make rpm
$ sudo dnf install ~/rpmbuild/RPMS/x86_64/itsa-*
```

### FreeBSD

Quick start.

Install dependencies

    $ sudo pkg install gmake jansson curl sqlite3 e2fsprogs-libuuid

Build libmtdac

```
$ git clone https://github.com/ac000/libmtdac.git
$ cd libmtdac
$ gmake
```

```
cd ..
```

```
$ git clone https://github.com/ac000/libac.git
$ cd libac
$ gmake
```

```
cd ..
```

and finally itsa itself

```
$ git clone https://github.com/ac000/itsa.git
$ cd itsa
$ CFLAGS=-I../../libac/src/include LIBS=-L../../libac/src gmake
```

The above _gmake_ commands will use GCC by default, to use clang, add CC=clang
to the gmake command, e.g

    $ gmake CC=clang

Then you can run it like

    $ LD_LIBRARY_PATH=../../libmtdac/src:../../libac/src ./itsa

# Using

itsa currently supports the following commands

```
Usage: itsa COMMAND [OPTIONS]

Commands
    init
    re-auth

    switch-business

    list-periods [<start> <end>]
    create-period [<start> <end>]
    update-period <period_id>
    update-annual-summary <tax_year>
    get-end-of-period-statement-obligations [<start> <end>]
    submit-end-of-period-statement <start> <end>
    submit-final-declaration <tax_year>
    list-calculations [tax_year]
    view-end-of-year-estimate
    add-savings-account
    view-savings-accounts [tax_year]
    amend-savings-account <tax_year>
```

It requires a little bit of config...

```
$ mkdir -p ~/.config/itsa
$ cp config.json.tmpl ~/.config/itsa/config.json
```

Set *production_api* accordingly.

Next you will need to run

```
$ itsa init
```

this need only be run once. Follow the instructions.

# Fraud Prevention Headers

It's important to point out that itsa will send various headers to HMRC with
various bits of information such as your IP addresses, MAC addresses,
OS username, a unique device ID.

# Environment variables

Currently there are two environment variables that can bet set to control
behaviour

### ITSA_LOG_LEVEL

This can be used to override the default log level (MTD\_OPT\_LOG\_ERR).

Currently recognised values are; *debug* & *info*

### VISUAL & EDITOR

For some things itsa will open an editor, to determine what editor to use,
itsa will first check the **VISUAL** environment variable and execute what
that's set to.

If that isn't set it will execute whatever **EDITOR** is set to.

If neither of those are set, itsa will default to **vi**.

### NO_COLOR

By default itsa will use colourised output. This can be disabled by setting
the **NO_COLOR** environment variable. Its value is unimportant (can be empty).

This can be overridden by [ITSA\_COLOR](#itsa_color)

### ITSA_COLOR

By default, itsa will use colourised output. If the above *NO\_COLOR*
environment variable is set then it won't.

*ITSA\_COLOR* can be used to either force the colourised output on or off
(regardless of the setting of *NO\_COLOR*).

It can be set to either *yes/true* or *no/false*

# License

itsa is licensed under the GNU General Public License (GPL) version 2

See *COPYING* in the repository root for details.

# Contributing

See *CodingStyle.md* & *Contributing.md*
