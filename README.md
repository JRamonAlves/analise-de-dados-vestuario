# analise-de-dados-vestuario

This repository contains `analise_vendas_vestuario.qmd` (Quarto) and supporting files used to render a data analysis of clothing sales.

This README documents how to reproduce the environment, fix the native build failures seen when rendering, and how to render the document locally.

## Quick summary

If your Quarto render fails with R package compilation errors (missing headers or compilers), you most likely need to install system development packages (C/C++/Fortran headers, CMake, and specific library `-dev` packages). The log from a failed render frequently shows missing headers like `curl/curl.h` or `libxml/tree.h`, or missing tools like `gfortran` and `cmake`.

## Recommended system packages (Ubuntu / Debian / WSL)

Run these commands in a `zsh` or `bash` terminal to install the common build dependencies needed by the R packages used in this project:

```bash
sudo apt update
sudo apt install -y build-essential gfortran pkg-config cmake \
  libcurl4-openssl-dev libxml2-dev libssl-dev libv8-dev libnlopt-dev \
  r-base-dev
```

Notes:

- `build-essential`: compiler toolchain (gcc/g++)
- `gfortran`: Fortran compiler required by many R packages
- `pkg-config`: used during package configuration to locate libraries
- `cmake`: required by packages like `nloptr`
- `libcurl4-openssl-dev`: provides `curl/curl.h` (fixes R package `curl`)
- `libxml2-dev`: provides `libxml/tree.h` (fixes R package `xml2`)
- `libv8-dev`: V8 engine headers used by R `V8` (may not be present on all distros)
- `libnlopt-dev`: NLopt library used by `nloptr`
- `r-base-dev`: meta-package providing common R build tools

If your system doesn't have `libv8-dev` in apt, see the Troubleshooting section below.

## Install required R packages

Open R (or use `Rscript`) and run:

```r
install.packages(c(
  "curl", "xml2", "V8", "juicyjuice", "gt", "gtsummary",
  "nloptr", "SparseM", "minqa", "corrplot", "skimr", "broom.helpers"
), repos = "https://cloud.r-project.org")
```

You can also run this from the shell:

```bash
Rscript -e 'install.packages(c("curl","xml2","V8","juicyjuice","gt","gtsummary","nloptr","SparseM","minqa","corrplot","skimr","broom.helpers"), repos="https://cloud.r-project.org")'
```

If you prefer to let Quarto install missing packages during render, you can skip explicit installs and run `quarto render` — but installing system deps first avoids repeated failures.

## Render the Quarto document

From the repository root:

```bash
quarto render analise_vendas_vestuario.qmd
# or from R
Rscript -e 'quarto::quarto_render("analise_vendas_vestuario.qmd")'
```

The output HTML will be `analise_vendas_vestuario.html`.

## Verify system tools are available

After installing packages, check these commands return versions (no errors):

```bash
gfortran --version
cmake --version
pkg-config --version
```

In R, test loading the packages that failed previously:

```r
library(curl)
library(xml2)
library(V8)
library(gt)
```

If these `library()` calls succeed, the main issues are resolved.

## Common errors in the render log and one-line fixes

- `fatal error: curl/curl.h: No such file or directory` → `sudo apt install libcurl4-openssl-dev`
- `fatal error: libxml/tree.h: No such file or directory` → `sudo apt install libxml2-dev`
- `sh: 1: gfortran: not found` → `sudo apt install gfortran` (or `r-base-dev`)
- `CMake was not found on the PATH` → `sudo apt install cmake`
- `ERROR: dependency ‘curl’ is not available for package ‘V8’` → fix `curl` first (see above), then re-install `V8`

## Troubleshooting and edge-cases

1. libv8 / V8 package problems

   - `libv8-dev` is not always available on all Ubuntu releases or may be named differently.
   - Workarounds:
     - Install Node / libnode headers: `sudo apt install nodejs libnode-dev`
     - Use a pre-built binary of `V8` if provided by your distribution or CRAN (rare on Linux)
     - Use Docker or conda (see below) to create an environment with the required library

2. No `sudo` or no root access

   - Ask your system administrator to install the above system packages, or
   - Use Docker (recommended) or `conda` to produce an isolated environment without changing the system.

3. Network / CRAN mirror issues

   - If downloads fail, pass a different CRAN mirror or check proxy/firewall settings.

4. Still failing after installing system deps
   - Confirm `pkg-config` can see the libraries: `pkg-config --modversion libcurl` and `pkg-config --modversion libxml-2.0`.
   - Check header locations with `dpkg -L libcurl4-openssl-dev | grep curl.h`.
   - Show the full R build error and paste it into an issue or chat for further debugging.
