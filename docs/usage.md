# nf-core/chipseq Usage

## Table of contents

* [Introduction](#general-nextflow-info)
* [Running the pipeline](#running-the-pipeline)
    * [Updating the pipeline](#updating-the-pipeline)
    * [Reproducibility](#reproducibility)
* [Main arguments](#main-arguments)
    * [`-profile`](#-profile-single-dash)
        * [`awsbatch`](#awsbatch)
        * [`conda`](#conda)
        * [`docker`](#docker)
        * [`singularity`](#singularity)
        * [`test`](#test)
    * [`--reads`](#--reads)
* [Generic arguments](#generic-arguments)
    * [`--singleEnd`](#--singleEnd)
    * [`--macsconfig`](#--macsconfig)
    * [`--allow_multi_align`](#--allow_multi_align)
    * [`--saveAlignedIntermediates`](#--saveAlignedIntermediates)
    * [`--skipDupRemoval`](#--skipDupRemoval)
    * [`--seqCenter`](#--seqCenter)
    * [`--fingerprintBins`](#--fingerprintBins)
    * [`--broad`](#--broad)
    * [`--macsgsize`](#--macsgsize)
    * [`--saturation`](#--saturation)
    * [`--extendReadsLen`](#--extendReadsLen)
* [Reference genomes](#reference-genomes)
    * [`--genome`](#--genome)
    * [`--fasta`](#--fasta)
    * [`--bwa_index`](#--bwa_index)
    * [`--largeRef`](#--largeRef)
    * [`--gtf`](#--gtf)
    * [`--bed`](#--bed)
    * [`--blacklist`](#--blacklist)
    * [`--blacklist_filtering`](#--blacklist_filtering)
    * [`--saveReference`](#--saveReference)
    * [`--igenomesIgnore`](#--igenomesignore)
* [Adapter trimming](#adapter-trimming)
    * [`--notrim`](#--notrim)
    * [`--saveTrimmed`](#--saveTrimmed)
* [Job resources](#job-resources)
    * [Automatic resubmission](#automatic-resubmission)
    * [Custom resource requests](#custom-resource-requests)
* [AWS batch specific parameters](#aws-batch-specific-parameters)
    * [`--awsqueue`](#--awsqueue)
    * [`--awsregion`](#--awsregion)
* [Other command line parameters](#other-command-line-parameters)
    * [`--outdir`](#--outdir)
    * [`--email`](#--email)
    * [`-name`](#-name-single-dash)
    * [`-resume`](#-resume-single-dash)
    * [`-c`](#-c-single-dash)
    * [`--custom_config_version`](#--custom_config_version)
    * [`--max_memory`](#--max_memory)
    * [`--max_time`](#--max_time)
    * [`--max_cpus`](#--max_cpus)
    * [`--plaintext_email`](#--plaintext_email)
    * [`--monochrome_logs`](#--monochrome_logs)
    * [`--multiqc_config`](#--multiqc_config)

## General Nextflow info
Nextflow handles job submissions on SLURM or other environments, and supervises running the jobs. Thus the Nextflow process must run until the pipeline is finished. We recommend that you put the process running in the background through `screen` / `tmux` or similar tool. Alternatively you can run nextflow within a cluster job submitted your job scheduler.

It is recommended to limit the Nextflow Java virtual machines memory. We recommend adding the following line to your environment (typically in `~/.bashrc` or `~./bash_profile`):

```bash
NXF_OPTS='-Xms1g -Xmx4g'
```

## Running the pipeline
The typical command for running the pipeline is as follows:
```bash
nextflow run nf-core/chipseq --reads '*_R{1,2}.fastq.gz' --genome GRCh37 --macsconfig 'macssetup.config' -profile docker
```

This will launch the pipeline with the `docker` configuration profile. See below for more information about profiles.

Note that the pipeline will create the following files in your working directory:

```bash
work            # Directory containing the nextflow working files
results         # Finished results (configurable, see below)
.nextflow_log   # Log file from Nextflow
# Other nextflow hidden files, eg. history of pipeline runs and old logs.
```

### Updating the pipeline
When you run the above command, Nextflow automatically pulls the pipeline code from GitHub and stores it as a cached version. When running the pipeline after this, it will always use the cached version if available - even if the pipeline has been updated since. To make sure that you're running the latest version of the pipeline, make sure that you regularly update the cached version of the pipeline:

```bash
nextflow pull nf-core/chipseq
```

### Reproducibility
It's a good idea to specify a pipeline version when running the pipeline on your data. This ensures that a specific version of the pipeline code and software are used when you run your pipeline. If you keep using the same tag, you'll be running the same version of the pipeline, even if there have been changes to the code since.

First, go to the [nf-core/chipseq releases page](https://github.com/nf-core/chipseq/releases) and find the latest version number - numeric only (eg. `1.0`). Then specify this when running the pipeline with `-r` (one hyphen) - eg. `-r 1.0`.

This version number will be logged in reports when you run the pipeline, so that you'll know what you used when youook back in the future.

## Main arguments

### `-profile`
Use this parameter to choose a configuration profile. Profiles can give configuration presets for different compute environments. Note that multiple profiles can be loaded, for example: `-profile docker` - the order of arguments is important!

If `-profile` is not specified at all the pipeline will be run locally and expects all software to be installed and available on the `PATH`.

* `awsbatch`
    * A generic configuration profile to be used with AWS Batch.
* `conda`
    * A generic configuration profile to be used with [conda](https://conda.io/docs/)
    * Pulls most software from [Bioconda](https://bioconda.github.io/)
* `docker`
    * A generic configuration profile to be used with [Docker](http://docker.com/)
    * Pulls software from dockerhub: [`nfcore/chipseq`](http://hub.docker.com/r/nfcore/chipseq/)
* `singularity`
    * A generic configuration profile to be used with [Singularity](http://singularity.lbl.gov/)
    * Pulls software from singularity-hub
* `test`
    * A profile with a complete configuration for automated testing
    * Includes links to test data so needs no other parameters

### `--reads`
Location of the input FastQ files:

```bash
 --reads 'path/to/data/sample_*_{1,2}.fastq'
```

**NB: Must be enclosed in quotes!**

Note that the `{1,2}` parentheses are required to specify paired end data. The file path should be in quotation marks to prevent shell glob expansion.

If left unspecified, the pipeline will assume that the data is in a directory called `data` in the working directory (`data/*{1,2}.fastq.gz`).

## Generic arguments

### `--singleEnd`
By default, the pipeline expects paired-end data. If you have single-end data, specify `--singleEnd` on the command line when you launch the pipeline. A normal glob pattern, enclosed in quotation marks, can then be used for `--reads`. For example: `--singleEnd --reads '*.fastq'`

It is not possible to run a mixture of single-end and paired-end files in one run.

### `--macsconfig`
The setup file for peak calling using MACS.

Default: `data/macsconfig`

Format:
```
ChIPSampleID1,CtrlSampleID1,AnalysisID1
ChIPSampleID2,CtrlSampleID2,AnalysisID2
ChIPSampleID3,,AnalysisID3
```

1. Column 1: ChIP sample name
    * Typically the file **base name**, shared between both reads _(no file type extension)_
    * _eg._ for `chip_sample_1_R1.fastq.gz` and `chip_sample_1_R2.fastq.gz`, enter `chip_sample_1`
2. Column 2: Control sample name
    * Typically the input file **base name**, shared between both reads _(no file type extension)_
    * _eg._ for `chip_input_R1.fastq.gz` and `chip_input_R2.fastq.gz`, enter `chip_input`
3. Column 3: Analysis ID
    * The analysis ID. Used for the output directory name.
    * Should be unique for each sample / line in the file.

For single-sample peaking calling without a control sample, leave the second column blank
(control sample name).

### `--allow_multi_align`
Specifying `--allow_multi_align` will turn off the filtering of secondary alignments and unmapped reads from the BWA output file. Without this option, only primary alignments will be retained.

### `--saveAlignedIntermediates`
By default, intermediate BAM files will not be saved. The final BAM files created
after the Picard MarkDuplicates step are always saved. Set to true to also copy out BAM
files from BWA and sorting steps.

### `--skipDupRemoval`
By default duplicate reads will be removed with picard. With this flag on this pipeline will skip duplicate removal and use raw BAM files for downstream analysis.

### `--seqCenter`
Text about sequencing center which will be added in the header of output bam files.

### `--fingerprintBins`
Number of genomic bins to use when calculating the deepTools Fingerprint plot.
Larger numbers will give a smoother profile, but take longer to run.

Default: `50000`

### `--broad`
Run MACS with the `--broad` flag. With this flag on, MACS will try to composite broad regions in BED12 ( a gene-model-like format ) by putting nearby highly enriched regions into a broad region with loose cutoff. The broad region is controlled by the default qvalue cutoff `0.1`.

### `--macsgsize`
Effective genome size which is used for the option `--gsize` in MACS. Should be in the format "2.1e9". See [`conf/igenomes.config`](conf/igenomes.config) for the predefined values of all supported reference genomes. This value is the mappable genome size or effective genome size which is defined as the genome size which can be sequenced. Because of the repetitive features on the chromsomes, the actual mappable genome size will be smaller than the original size, about 90% or 70% of the genome size.

### `--saturation`
Run saturation analysis by sub-sampling the sequence reads of ChIP sample from 10% to 100% with 10% interval, then calling ChIP-seq peaks with MACS. For one test there will be 10 sets of ChIP-seq peaks. A summary file (.CSV) will be provided with the numbers of ChIP-seq peaks.

### `--extendReadsLen`
Number of base pairs to extend the reads for the deepTools analysis. This number should be
based on the length of your reads and the expected fragment length.

Default: `100`

## Reference Genomes

The pipeline config files come bundled with paths to the illumina iGenomes reference index files. If running with docker or AWS, the configuration is set up to use the [AWS-iGenomes](https://ewels.github.io/AWS-iGenomes/) resource.

### `--genome` (using iGenomes)
There are 31 different species supported in the iGenomes references. To run the pipeline, you must specify which to use with the `--genome` flag.

_Currently only the human and mouse genomes are fully supported by this pipeline. For other genomes `MACS` and `NGSplot` will not run!_

You can find the keys to specify the genomes in the [`iGenomes config file`](../conf/igenomes.config). Common genomes that are supported are:

* Human
  * `--genome GRCh37`
* Mouse
  * `--genome GRCm38`
* _Drosophila_
  * `--genome BDGP6`
* _S. cerevisiae_
  * `--genome 'R64-1-1'`

> There are numerous others - check the config file for more.

Note that you can use the same configuration setup to save sets of reference files for your own use, even if they are not part of the iGenomes resource. See the [Nextflow documentation](https://www.nextflow.io/docs/latest/config.html) for instructions on where to save such a file.

The syntax for this reference configuration is as follows:

```nextflow
params {
  genomes {
    'GRCh37' {
      fasta   = '<path to the genome fasta file>' // Used if no star index given
    }
    // Any number of additional genomes, key is used with --genome
  }
}
```

### `--fasta`
If you don't have a BWA index available, you can pass a FASTA file to the pipeline and a BWA index
will be generated for you. Combine with `--saveReference` to save for future runs.
```bash
--fasta '[path to FASTA file]'
```

### `--bwa_index`
Full path to an existing BWA index for your reference genome including the base name for the index.
```bash
--bwa_index '[directory containing BWA index]/genome.fa'
```

### `--largeRef`
Build BWA Index with the `bwtsw` flag for reference genomes larger than 2Gb in size (such as human). By default the `is` option will be used.

### `--gtf`
The full path to GTF file can be specified for annotating peaks. Note that the GTF file should be in the Ensembl format.
```bash
--gtf '[path to GTF file]'
```

### `--bed`
The full path to BED file for computing read distribution matrix for deepTools. Note that the BED file should be in the Ensembl format.
```bash
--bed '[path to BED file]'
```

### `--blacklist`
If you prefer, you can specify the full path to the blacklist regions (should be in .BED format) which will be filtered out from the called ChIP-seq peaks. Please note that `--blacklist_filtering` is required for using this option.
```bash
--blacklist_filtering --blacklist '[path to blacklisted regions]'
```

### `--blacklist_filtering`
Specifying this flag instructs the pipeline to use bundled ENCODE blacklist regions to filter out known blacklisted regions in the called ChIP-seq peaks. The following reference genome builds are supported:
* Human: `GRCh37`, `GRCh38`
* Mouse: `GRCm37`, `GRCm38`
* C.elegans: `WBcel235`
* D.melanogaster: `BDGP6`

### `--saveReference`
Supply this parameter to save any generated reference genome files to your results folder. These can then be used for future pipeline runs, reducing processing times.

### `--igenomesIgnore`
Do not load `igenomes.config` when running the pipeline. You may choose this option if you observe clashes between custom parameters and those supplied in `igenomes.config`.

## Adapter Trimming

The pipeline accepts a number of parameters to change how the trimming is done, according to your data type.
You can specify custom trimming parameters as follows:

* `--clip_r1 <NUMBER>`
  * Instructs Trim Galore to remove bp from the 5' end of read 1 (or single-end reads).
* `--clip_r2 <NUMBER>`
  * Instructs Trim Galore to remove bp from the 5' end of read 2 (paired-end reads only).
* `--three_prime_clip_r1 <NUMBER>`
  * Instructs Trim Galore to remove bp from the 3' end of read 1 _AFTER_ adapter/quality trimming has been
* `--three_prime_clip_r2 <NUMBER>`
  * Instructs Trim Galore to re move bp from the 3' end of read 2 _AFTER_ adapter/quality trimming has been performed.

### `--notrim`
Specifying `--notrim` will skip the adapter trimming step. Use this if your input FastQ files have already been trimmed outside of the workflow or if you're very confident that there is no adapter contamination in your data.

### `--saveTrimmed`
By default, trimmed FastQ files will not be saved to the results directory. Specify this flag (or set to true in your config file) to copy these files to the results directory when complete.

## Job resources
### Automatic resubmission
Each step in the pipeline has a default set of requirements for number of CPUs, memory and time. For most of the steps in the pipeline, if the job exits with an error code of `143` (exceeded requested resources) it will automatically resubmit with higher requests (2 x original, then 3 x original). If it still fails after three times then the pipeline is stopped.

### Custom resource requests
Wherever process-specific requirements are set in the pipeline, the default value can be changed by creating a custom config file. See the files hosted at [`nf-core/configs`](https://github.com/nf-core/configs/tree/master/conf) for examples.

If you are likely to be running `nf-core` pipelines regularly it may be a good idea to request that your custom config file is uploaded to the `nf-core/configs` git repository. Before you do this please can you test that the config file works with your pipeline of choice using the `-c` parameter (see definition below). You can then create a pull request to the `nf-core/configs` repository with the addition of your config file, associated documentation file (see examples in [`nf-core/configs/docs`](https://github.com/nf-core/configs/tree/master/docs)), and amending [`nfcore_custom.config`](https://github.com/nf-core/configs/blob/master/nfcore_custom.config) to include your custom profile.

If you have any questions or issues please send us a message on [`Slack`](https://nf-core-invite.herokuapp.com/).

## AWS Batch specific parameters
Running the pipeline on AWS Batch requires a couple of specific parameters to be set according to your AWS Batch configuration. Please use the `-awsbatch` profile and then specify all of the following parameters.
### `--awsqueue`
The JobQueue that you intend to use on AWS Batch.
### `--awsregion`
The AWS region to run your job in. Default is set to `eu-west-1` but can be adjusted to your needs.

Please make sure to also set the `-w/--work-dir` and `--outdir` parameters to a S3 storage bucket of your choice - you'll get an error message notifying you if you didn't.

## Other command line parameters

### `--outdir`
The output directory where the results will be saved.

### `--email`
Set this parameter to your e-mail address to get a summary e-mail with details of the run sent to you when the workflow exits. If set in your user config file (`~/.nextflow/config`) then you don't need to specify this on the command line for every run.

### `-name`
Name for the pipeline run. If not specified, Nextflow will automatically generate a random mnemonic.

This is used in the MultiQC report (if not default) and in the summary HTML / e-mail (always).

**NB:** Single hyphen (core Nextflow option)

### `-resume`
Specify this when restarting a pipeline. Nextflow will used cached results from any pipeline steps where the inputs are the same, continuing from where it got to previously.

You can also supply a run name to resume a specific run: `-resume [run-name]`. Use the `nextflow log` command to show previous run names.

**NB:** Single hyphen (core Nextflow option)

### `-c`
Specify the path to a specific config file (this is a core NextFlow command).

**NB:** Single hyphen (core Nextflow option)

Note - you can use this to override pipeline defaults.

### `--custom_config_version`
Provide git commit id for custom Institutional configs hosted at `nf-core/configs`. This was implemented for reproducibility purposes. Default is set to `master`.

```bash
## Download and use config file with following git commid id
--custom_config_version d52db660777c4bf36546ddb188ec530c3ada1b96
```

### `--max_memory`
Use to set a top-limit for the default memory requirement for each process.
Should be a string in the format integer-unit. eg. `--max_memory '8.GB'`

### `--max_time`
Use to set a top-limit for the default time requirement for each process.
Should be a string in the format integer-unit. eg. `--max_time '2.h'`

### `--max_cpus`
Use to set a top-limit for the default CPU requirement for each process.
Should be a string in the format integer-unit. eg. `--max_cpus 1`

### `--plaintext_email`
Set to receive plain-text e-mails instead of HTML formatted.

### `--monochrome_logs`
Set to disable colourful command line output and live life in monochrome.

### `--multiqc_config`
Specify a path to a custom MultiQC configuration file.