# MIRUReader

## Description

Identify 24-locus MIRU-VNTR for _Mycobacterium tuberculosis_ complex (MTBC) directly from long reads generated by Oxford Nanopore Technologies (ONT) and Pacific Biosciences (PacBio). Also work on assembled genome.

## Requirements

- Linux
- primersearch from [EMBOSS](http://emboss.sourceforge.net/download/)
  - install from the official website or
  - install via conda `conda install -c bioconda emboss`
  - Ensure the primersearch command is in your device's environment path, where primersearch program can be executed directly by typing `primersearch` on the commandline
- [_pandas_](https://pandas.pydata.org/)
  - can be installed via conda `conda install pandas` or via PyPI `pip install pandas`
- [_statistics_](https://pypi.org/project/statistics/)
  - can be installed via PyPI `pip install statistics`

## Installation

`git clone https://github.com/phglab/MIRUReader.git` or `git clone https://github.com/MG-IiSGM/MIRUReader` (for this modified version)

## Change log

#### 03/07/2024

- Added different depth and frequency parameters to flag potential suboptimal alleles.
- Updated interpretation documentation to the README

#### 13/09/2019

- Added a check to ensure primersearch is executable prior to MIRUReader program execution
- Updated documentation to the README

#### 04/07/2019

- Update output format for option '--details'.

#### 14/06/2019

- Auto convert fastq to fasta.

## Usage example

For one sample analysis:

```
python /your/path/to/MIRUReader.py -r sample.fasta -p sampleID > miru.txt
```

For multiple samples analysis:

1. Create a mapping file (mappingFile.txt) that looks like:

   sample_001.fasta sample_001 \
   sample_002.fasta sample_002 \
   ...

2. Then run the program:

```
cat mappingFile.txt | while read -a line; do python /your/path/to/MIRUReader.py -r ${line[0]} -p ${line[1]}; done > miru.multiple.txt
```

## Output example

```
sample_prefix   0154    0424    0577    0580    0802    0960    1644    1955    2059    2163b   2165    2347    2401    2461    2531    2687    2996    3007    3171    3192    3690    4052    4156    4348
sample_001      2       4       4       2       3       3       3       2       2       5       4       4       4       2       5       1       6       3       3       5       3       7       2       3
```

Notes:

- The program is compatible to Python 2 and Python 3.
- Accepted reads file format includes '.fastq', '.fastq.gz', '.fasta', and '.fasta.gz'.
- The program output is a tab-delimited plain text which can be copied to or opened in Excel spreadsheet.

## Full usage

| Main options      | Description                                                                                                                                                    |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| -r READS          | Input reads file in fastq/fasta format, can be gzipped or not gzipped                                                                                          |
| -p PREFIX         | Sample ID required for naming output file.                                                                                                                     |
| --table TABLE     | Allele calling table, default is MIRU_table. Can be user-defined in fixed format. However, providing custom allele calling table for other VNTR is not tested. |
| --primers PRIMERS | Primers sequences, default is MIRU_primers. Can be user-defined in fixed format.                                                                               |

| Optional options | Description                                                                                                                                                                     |
| ---------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| --amplicons      | Use output from primersearch ("prefix.18.primersearch.out") and summarize MIRU profile directly.                                                                                |
| --nofasta        | Delete fasta file generated if your input read is in fastq format.                                                                                                              |
| --mismatch       | Allowed percent mismatch. Default: 18                                                                                                                                           |
| --min_amplicons  | Minimum number of amplicons required for a reliable result. Below this threshold, the program returns "Warning 1" for low coverage. Default: 3                                  |
| --freq           | Minimum frequency required for a reliable result. Below this threshold, the program returns "Warning 2" for an unfixed allele. Default: 0.6                                     |
| --amplicon_freq  | Number of amplicons required for reliable results with mixed alleles. Below this threshold, the program returns "Warning 2" for an unfixed allele. Default: 20 [Flag commented] |
| --amplicon_mode  | Number of amplicons required for reliable results. Below this threshold, the program returns "Warning 3" for a possible polyclonal allele with low coverage. Default: 10        |

| Interpretation | Description                                                                                                                                             |
| -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Warning 1      | Low coverage (There are fewer than 3 amplicons (default), or the value indicated by --min_amplicons)                                                    |
| Warning 2      | Unfixed allele (When the majority value has a frequency of less than 0.6 (default) at that locus. [Supported by 20 or fewer amplicons, flag commented]) |
| Warning 3      | Possible polyclonal - Low coverage (When there are 2 modes with values of the same majority frequency and fewer than 10 (default) amplicons)            |
| Warning 4      | Possible polyclonal (There are 2 modes and more than 10 amplicons (default) validating the locus.)                                                      |

All warnings must be taken into account due to low coverage or frequencies, and they should be inspected manually or even repeated.

## FAQ

1. **Why are there two MIRU allele calling tables (MIRU_table and MIRU_table_0580)?**

MIRU loci 0580 (MIRU_table_0580) consist of a different numbering system for determination of repeat numbers as compared to the other 23 MIRU locus (MIRU_table) for MTBC isolates.

## Troubleshooting

1. If an error message `OSError: primersearch is not found.` appears, please ensure your `primersearch` executable file is in your environment path (`echo $PATH`) and can be called directly.

2. If analyzing from a `.fasta assembly`, 'Warning 1' for low coverage will appear, as contigs are used and only a single fragment should support the locus region.
