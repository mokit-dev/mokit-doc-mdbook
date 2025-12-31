# mdbook Pages for MOKIT Doc

## Build and preview the documentation locally

Install `mdbook` first. You can use `cargo install mdbook` or go to the [mdbook release](https://github.com/rust-lang/mdBook/releases) to download the executables. Then,
```
# modify md in src
mdbook build
mdbook serve
# open your browser to visit http://localhost:3000
```

## Offline browsing

Download the released tar.gz of this repo, and do

```
tar xzvf mokit-doc-1.2.5rc19.tar.gz
cd mokit-doc-1.2.5rc19
firefox public/index.html # double-click it if using Windows
```

Most of the the links in doc in offline mode should be still valid, but I'm not very sure...

## Features

### Theme

<!--Use Nord Theme from [gbrlsnchs/mdBook-nord-template](https://github.com/gbrlsnchs/mdBook-nord-template).-->

### Note/Warning

<!--See [blocks for note](https://jeanwsr.gitlab.io/mokit-doc-mdbook/chap6.html#blocks-for-notewarning).-->





