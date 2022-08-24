# Eget: easy pre-built binary installation

[![Go Report Card](https://goreportcard.com/badge/github.com/zyedidia/eget)](https://goreportcard.com/report/github.com/zyedidia/eget)
[![Release](https://img.shields.io/github/release/zyedidia/eget.svg?label=Release)](https://github.com/zyedidia/eget/releases)
[![MIT License](https://img.shields.io/badge/license-MIT-blue.svg)](https://github.com/zyedidia/eget/blob/master/LICENSE)

**Eget** is the best way to easily get pre-built binaries for your favorite
tools. It downloads and extracts pre-built binaries from releases on GitHub. To
use it, provide a repository and Eget will search through the assets from the
latest release in an attempt to find a suitable prebuilt binary for your
system. If one is found, the asset will be downloaded and Eget will extract the
binary to the current directory. Eget should only be used for installing
simple, static prebuilt binaries, where the extracted binary is all that is
needed for installation. For more complex installation, you may use the
`--download-only` option, and perform extraction manually.

![Eget Demo](https://github.com/zyedidia/blobs/blob/master/eget-demo.gif)

For software maintainers, if you provide prebuilt binaries on GitHub, you can
list `eget` as a one-line method for users to install your software.

Eget has a number of detection mechanisms and should work out-of-the-box with
most software that is distributed via single binaries on GitHub releases. First
try using Eget on your software, it may already just work. Otherwise, see the
FAQ for a clear set of rules to make your software compatible with Eget.

For more in-depth documentation, see [DOCS.md](DOCS.md).

# Examples

```
eget zyedidia/micro --tag nightly
eget jgm/pandoc --to /usr/local/bin
eget junegunn/fzf
eget neovim/neovim
eget ogham/exa --asset ^musl
eget --system darwin/amd64 sharkdp/fd
eget BurntSushi/ripgrep
eget -f eget.1 zyedidia/eget
eget zachjs/sv2v
eget https://go.dev/dl/go1.17.5.linux-amd64.tar.gz --file go --to ~/go1.17.5
eget --all --file '*' ActivityWatch/activitywatch
```

# How to get Eget

Before you can get anything, you have to get Eget. If you already have Eget and want to upgrade, use `eget zyedidia/eget`.

### Quick-install script

```
curl -o eget.sh https://zyedidia.github.io/eget.sh
shasum -a 256 eget.sh # verify with hash below
bash eget.sh
```

Or alternatively (less secure):

```
curl https://zyedidia.github.io/eget.sh | sh
```

You can then place the downloaded binary in a location on your `$PATH` such as `/usr/local/bin`.

To verify the script, the sha256 checksum is `0e64b8a3c13f531da005096cc364ac77835bda54276fedef6c62f3dbdc1ee919` (use `shasum -a 256 eget.sh` after downloading the script).

One of the reasons to use eget is to avoid running curl into bash, but unfortunately you can't eget eget until you have eget.

### Homebrew

```
brew install eget
```

### Pre-built binaries

Pre-built binaries are available on the [releases](https://github.com/zyedidia/eget/releases) page.

### From source

Install the latest released version:

```
go install github.com/zyedidia/eget@latest
```

or install from HEAD:

```
git clone https://github.com/zyedidia/eget
cd eget
make build # or go build (produces incomplete version information)
```

A man page can be generated by cloning the repository and running `make eget.1`
(requires pandoc). You can also use `eget` to download the man page: `eget -f eget.1 zyedidia/eget`.

# Usage

The `TARGET` argument passed to Eget should either be a GitHub repository,
formatted as `user/repo`, in which case Eget will search the release assets, a
direct URL, in which case Eget will directly download and extract from the
given URL, or a local file, in which case Eget will extract directly from the
local file.

If Eget downloads an asset called `xxx` and there also exists an asset called
`xxx.sha256` or `xxx.sha256sum`, Eget will automatically verify that the
SHA-256 checksum of the downloaded asset matches the one contained in that
file, and abort installation if a mismatch occurs.

When installing an executable, Eget will place it in the current directory by
default. If the environment variable `EGET_BIN` is non-empty, Eget will
place the executable in that directory.

Directories can also be specified as files to extract, and all files within
them will be extracted. For example:

```
eget https://go.dev/dl/go1.17.5.linux-amd64.tar.gz --file go --to ~/go1.17.5
```

GitHub limits API requests to 60 per hour for unauthenticated users. If you
would like to perform more requests (up to 5,000 per hour), you can set up a
personal access token and assign it to the environment variable `GITHUB_TOKEN`
when running Eget. Eget will read this variable and send the token as
authorization with requests to GitHub.

```
Usage:
  eget [OPTIONS] TARGET

Application Options:
  -t, --tag=           tagged release to use instead of latest
      --pre-release    include pre-releases when fetching the latest version
      --source         download the source code for the target repo instead of a release
      --to=            move to given location after extracting
  -s, --system=        target system to download for (use "all" for all choices)
  -f, --file=          glob to select files for extraction
      --all            extract all candidate files
  -q, --quiet          only print essential output
  -d, --download-only  stop after downloading the asset (no extraction)
      --upgrade-only   only download if release is more recent than current version
  -a, --asset=         download a specific asset containing the given string; can be specified
                       multiple times for additional filtering; use ^ for anti-match
      --sha256         show the SHA-256 hash of the downloaded asset
      --verify-sha256= verify the downloaded asset checksum against the one provided
      --rate           show GitHub API rate limiting information
  -r, --remove         remove the given file from $EGET_BIN or the current directory
  -v, --version        show version information
  -h, --help           show this help message
```

# FAQ

### How is this different from a package manager?

Eget only downloads pre-built binaries uploaded to GitHub by the developers of
the repository. It does not maintain a central list of packages, nor does it do
any dependency management. Eget does not "install" executables by placing them
in system-wide directories (such as `/usr/local/bin`) unless instructed, and it
does not maintain a registry for uninstallation. Eget works best for installing
software that comes as a single binary with no additional files needed (CLI
tools made in Go, Rust, or Haskell tend to fit this description).

### Is this secure?

Eget does not run any downloaded code -- it just finds executables from GitHub
releases and downloads/extracts them. If you trust the code you are downloading
(i.e. if you trust downloading pre-built binaries from GitHub) then using Eget
is perfectly safe. If Eget finds a matching asset ending in `.sha256` or
`.sha256sum`, the SHA-256 checksum of your download will be automatically
verified. You can also use the `--sha256` or `--verify-sha256` options to
manually verify the SHA-256 checksums of your downloads (checksums are provided
in an alternative manner by your download source).

### Does this work only for GitHub repositories?

At the moment Eget supports searching GitHub releases, direct URLs, and local
files. If you provide a direct URL instead of a GitHub repository, Eget will
skip the detection phase and download directly from the given URL. If you
provide a local file, Eget will skip detection and download and just perform
extraction from the local file.

### How can I make my software compatible with Eget?

Eget should work out-of-the-box with many methods for releasing software, and
does not require that you build your release process for Eget in particular.
However, here are some rules that will guarantee compatibility with Eget.

* Provide your pre-built binaries as GitHub release assets.
* Format the system name as `OS_Arch` and include it in every pre-built binary
  name. Supported OSes are `darwin`/`macos`, `windows`, `linux`, `netbsd`,
  `openbsd`, `freebsd`, `android`, `illumos`, `solaris`, `plan9`. Supported
  architectures are `amd64`, `i386`, `arm`, `arm64`, `riscv64`.
* If desired, include `*.sha256` files for each asset, containing the SHA-256
  checksum of each asset. These checksums will be automatically verified by
  Eget.
* Include only a single executable or appimage per system in each release archive.
* Use `.tar.gz`, `.tar.bz2`, `.tar.xz`, `.tar`, or `.zip` for archives. You may
  also directly upload the executable without an archive, or a compressed
  executable ending in `.gz`, `.bz2`, or `.xz`.

# Contributing

If you find a bug, have a suggestion, or something else, please open an issue
for discussion. I am sometimes prone to leaving pull requests unmerged, so
please double check with me before investing lots of time into implementing a
pull request. See [DOCS.md](DOCS.md) for more in-depth documentation.
