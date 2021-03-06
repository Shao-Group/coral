[![Anaconda-Server Badge](https://anaconda.org/bioconda/rnabridge-align/badges/installer/conda.svg)](https://anaconda.org/bioconda/rnabridge-align)
[![Anaconda-Server Badge](https://anaconda.org/bioconda/rnabridge-align/badges/downloads.svg)](https://anaconda.org/bioconda/rnabridge-align)

# Overview
rnabridge-align implements an efficient algorithm to bridge paire-end RNA-seq reads, i.e.,
to determine the alignment of full fragments given the alignment of two mate ends.
Its sister tool, [rnabridge-denovo](https://github.com/Shao-Group/rnabridge-denovo), 
determines the sequences of full fragments given the sequences of paired-end reads.
See [rnabridge-test](https://github.com/Shao-Group/rnabridge-test) for the evaluation of both tools.

# Release
Latest release of rnabridge-align is [v1.0.1](https://github.com/Shao-Group/rnabridge-align/releases/tag/v1.0.1).

# Installation
Download the source code of rnabridge-align from
[here](https://github.com/Shao-Group/rnabridge-align/releases/download/v1.0.1/rnabridge-align-1.0.1.tar.gz).
rnabridge-align uses additional libraries of Boost and htslib. 
If they have not been installed in your system, you first
need to download and install them. You might also need to
export the runtime library path to certain environmental
variable (for example, `LD_LIBRARY_PATH`, for most linux distributions).
After install these dependencies, you then compile the source code of rnabridge-align.
If some of the above dependencies are not installed to the default system 
directories (for example, `/usr/local`, for most linux distributions),
their corresponding installing paths should be specified to `configure` of rnabridge-align.

## Download Boost
If Boost has not been downloaded/installed, download Boost
[(license)](http://www.boost.org/LICENSE_1_0.txt) from (http://www.boost.org).
Uncompress it somewhere (compiling and installing are not necessary).

## Install htslib
If htslib has not been installed, download htslib 
[(license)](https://github.com/samtools/htslib/blob/develop/LICENSE)
from (http://www.htslib.org/) with version 1.5 or higher.
(Note that htslib relies on zlib. So if zlib has not been installed in your system,
you need to install zlib first.) 

Use the following commands to build htslib:
```
./configure --disable-bz2 --disable-lzma --disable-gcs --disable-s3 --enable-libcurl=no
make
make install
```
The default installation location of htslib is `/usr/lib`.
If you would install it to a different location, replace the above `configure` line with
the following (by adding `--prefix=/path/to/your/htslib` to the end):
```
./configure --disable-bz2 --disable-lzma --disable-gcs --disable-s3 --enable-libcurl=no --prefix=/path/to/your/htslib
```
In this case, you also need to export the runtime library path (note that there
is an additional `lib` following the installation path):
```
export LD_LIBRARY_PATH=/path/to/your/htslib/lib:$LD_LIBRARY_PATH
```

## Compile rnabridge-align

Use the following to compile rnabridge-align:
```
./configure --with-htslib=/path/to/your/htslib --with-boost=/path/to/your/boost
make
```

If some of the dependencies are installed in the default system directory (for example, `/usr/lib`),
then the corresponding `--with-` option might not be necessary.
The executable file `rnabridge-align` will appear at `src/rnabridge-align`.


# Usage

The usage of `rnabridge-align` is:
```
./rnabridge-align -i <input.bam> -o <output.bam> [-r reference.gtf] [options]
```

The `input.bam` is the read alignment file generated by some RNA-seq aligner, (for example, TopHat2, STAR, or HISAT2).
Make sure that it is sorted; otherwise run `samtools` to sort it:
```
samtools sort input.bam > input.sort.bam
```

The alignment of entire fragments shall be written to `output.bam`.

rnabridge-align also supports making use the reference transcriptome to improve bridging accuracy.
The reference transcriptome can be provided with `-r reference.gtf`.

rnabridge-align support the following parameters. 

 Parameters | Default Value | Description
 ------------------------- | ------------- | ----------
 --help  | | print usage of rnabridge-align and exit
 --version | | print version of rnabridge-align and exit
 --preview | | show the inferred `library_type` and `fragment-length-range` and exit
 --library_type               | empty | chosen from {empty, unstranded, first, second} (see below)
 --min_bridging_score | 0.5 | the minimized bottleneck weight in bridging path 
 --dp_solution_size | 10 | candidate number of bridgign paths
 --dp_stack_size | 5 | number of weights maintained for each bridging path
 --max_clustring_flank | 30 | maximized basepair difference for being in an equivalent class
 --flank_tiny_length | 10 | maximized length for reconsidering error correction
 --flank_tiny_ratio | 0.4 | maximized ratio for reconsidering error correction
 --min_splice_bundary_hits    | 1 | the minimum number of spliced reads required to support a junction
 --max_num_cigar              | 1000 | ignore reads with CIGAR size larger than this value

`--library_type` is highly recommended to provide. The `unstranded`, `first`, and `second`
correspond to `fr-unstranded`, `fr-firststrand`, and `fr-secondstrand` used in standard Illumina
sequencing libraries. If none of them is given, i.e., it is `empty` by default, then rnabridge-align
will try to infer the `library_type` by itself (see `--preview`). Notice that such inference is based
on the `XS` tag stored in the input `bam` file. If the input `bam` file do not contain `XS` tag,
then it is essential to provide the `library_type` to rnabridge-align. You can try `--preview` to see
the inferred `library_type`.

