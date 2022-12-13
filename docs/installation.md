# Installation

## Homebrew tap
 
The easiest way to install `dbsnapper` is via [Homebrew](https://brew.sh/)

```sh
brew install dbsnapper/tap/dbsnapper
```

## deb, rpm, and apk packages

Download the `.deb`, `.rpm`, or `.apk` packages from the [releases page][releases] and install them with the appropriate tools.

```sh title="Example installation via dpkg"
# Download the release
TAG=1.2.1
wget https://github.com/dbsnapper/dbsnapper/releases/download/v"$TAG"/dbsnapper_"$TAG"_Linux_amd64.deb 

## Install with dpkg
dpkg -i dbsnapper_"$TAG"_Linux_amd64.deb
```

[releases]: https://github.com/dbsnapper/dbsnapper/releases