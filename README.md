# ![nf-core/nfvibrio](docs/images/nf-core-nfvibrio_logo_light.png#gh-light-mode-only) ![nf-core/nfvibrio](docs/images/nf-core-nfvibrio_logo_dark.png#gh-dark-mode-only)

<!-- [![AWS CI](https://img.shields.io/badge/CI%20tests-full%20size-FF9900?labelColor=000000&logo=Amazon%20AWS)](https://nf-co.re/nfvibrio/results)[![Cite with Zenodo](http://img.shields.io/badge/DOI-10.5281/zenodo.XXXXXXX-1073c8?labelColor=000000)](https://doi.org/10.5281/zenodo.XXXXXXX) -->

[![Nextflow](https://img.shields.io/badge/nextflow%20DSL2-%E2%89%A522.10.1-23aa62.svg)](https://www.nextflow.io/)
[![run with conda](http://img.shields.io/badge/run%20with-conda-3EB049?labelColor=000000&logo=anaconda)](https://docs.conda.io/en/latest/)
[![run with docker](https://img.shields.io/badge/run%20with-docker-0db7ed?labelColor=000000&logo=docker)](https://www.docker.com/)
[![run with singularity](https://img.shields.io/badge/run%20with-singularity-1d355c.svg?labelColor=000000)](https://sylabs.io/docs/)
[![Launch on Nextflow Tower](https://img.shields.io/badge/Launch%20%F0%9F%9A%80-Nextflow%20Tower-%234256e7)](https://tower.nf/launch?pipeline=https://github.com/nf-core/nfvibrio)

<!-- [![Get help on Slack](http://img.shields.io/badge/slack-nf--core%20%23nfvibrio-4A154B?labelColor=000000&logo=slack)](https://nfcore.slack.com/channels/
nfvibrio)[![Follow on Twitter](http://img.shields.io/badge/twitter-%40nf__core-1DA1F2?labelColor=000000&logo=twitter)](https://twitter.com/nf_core)[![Watch on YouTube](http://img.shields.io/badge/youtube-nf--core-FF0000?labelColor=000000&logo=youtube)](https://www.youtube.com/c/nf-core) -->

## Introduction

<!-- TODO nf-core: Write a 1-2 sentence summary of what data the pipeline is for and what it does -->

**nf-core/nfvibrio** is a bioinformatics best-practice analysis pipeline for _Vibrio parahaemolyticus_ analysis. This workflow is under active development and updates. If you are using it (thank you!) remember to keep your fork updated by regularly syncing it.

The pipeline is built using [Nextflow](https://www.nextflow.io), a workflow tool to run tasks across multiple compute infrastructures in a very portable manner. It uses Docker/Singularity containers making installation trivial and results highly reproducible. The [Nextflow DSL2](https://www.nextflow.io/docs/latest/dsl2.html) implementation of this pipeline uses one container per process which makes it much easier to maintain and update software dependencies. <!-- Where possible, these processes have been submitted to and installed from [nf-core/modules](https://github.com/nf-core/modules) in order to make them available to all nf-core pipelines, and to everyone within the Nextflow community! -->

<!-- TODO nf-core: Add full-sized test dataset and amend the paragraph below if applicable

On release, automated continuous integration tests run the pipeline on a full-sized dataset on the AWS cloud infrastructure. This ensures that the pipeline runs on AWS, has sensible resource allocation defaults set to run on real-world datasets, and permits the persistent storage of results to benchmark between pipeline releases and other analysis sources.The results obtained from the full-sized test can be viewed on the [nf-core website](https://nf-co.re/nfvibrio/results).
-->

## Pipeline summary

<!-- TODO nf-core: Fill in short bullet-pointed list of the default steps in the pipeline -->

1. Read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
2. Performing read QC ([`FASTP`](https://github.com/OpenGene/fastp))
3. Assemble quality controlled reads ([`SHOVILL`](https://github.com/tseemann/shovill))
   1. Estimating genome size by counting unqiue kmer-mers ([`KMC`](https://github.com/refresh-bio/KMC))
   2. Correct sequencing errors in reads ([`Lighter`](https://github.com/mourisl/Lighter))
   3. Merge paired-end reads ([`FLASH`](https://ccb.jhu.edu/software/FLASH/))
   4. Assemble with ([`SPAdes`](https://github.com/ablab/spades))/([`SKESA`](https://github.com/ncbi/SKESA))/([`Megahit`](https://github.com/voutcn/megahit)) with modified kmer range
   5. Correct minor assembly errors by mapping reads back to contigs ([`PILON`](https://github.com/broadinstitute/pilon))
4. Perform assembly QC ([`QUAST`](https://github.com/ablab/quast))
5. Annotate assembled contigs ([`PROKKA`](https://github.com/tseemann/prokka))
6. Find antimicrobial resistance genes and point mutations in assembled contigs ([`AMRFINDERPLUS`](https://github.com/ncbi/amr))
7. Scan contig files against traditional PubMLST typing schemes ([`MLST`](https://github.com/tseemann/mlst))
8. Present QC for raw reads ([`MULTIQC`](http://multiqc.info/))

## Quick Start

1. Install [`Nextflow`](https://www.nextflow.io/docs/latest/getstarted.html#installation) (`>=21.0.0`)

2. Install any of [`Docker`](https://docs.docker.com/engine/installation/), [`Singularity`](https://www.sylabs.io/guides/3.0/user-guide/) (you can follow [this tutorial](https://singularity-tutorial.github.io/01-installation/)), [`Podman`](https://podman.io/), [`Shifter`](https://nersc.gitlab.io/development/shifter/how-to-use/) or [`Charliecloud`](https://hpc.github.io/charliecloud/) for full pipeline reproducibility _(you can use [`Conda`](https://conda.io/miniconda.html) both to install Nextflow itself and also to manage software within pipelines. Please only use it within pipelines as a last resort; see [docs](https://nf-co.re/usage/configuration#basic-configuration-profiles))_.

3. Download the pipeline and test it on a minimal dataset with a single command:

   ```bash
   nextflow run nfvibrio/main.nf -profile <conda,singularity,YOURPROFILE>  --genome "VP01" --input sample_sheet.csv --outdir <OUTDIR> --assembler <skesa,megahit,spades>
   ```

   Note that some form of configuration will be needed so that Nextflow knows how to fetch the required software. This is usually done in the form of a config profile (`YOURPROFILE` in the example command above). You can chain multiple config profiles in a comma-separated string.

   > - The pipeline comes with config profiles called `docker`, `singularity`, `podman`, `shifter`, `charliecloud` and `conda` which instruct the pipeline to use the named tool for software management. For example, `-profile test,docker`.
   > - Please check [nf-core/configs](https://github.com/nf-core/configs#documentation) to see if a custom config file to run nf-core pipelines already exists for your Institute. If so, you can simply use `-profile <institute>` in your command. This will enable either `docker` or `singularity` and set the appropriate execution settings for your local compute environment.
   > - If you are using `singularity`, please use the [`nf-core download`](https://nf-co.re/tools/#downloading-pipelines-for-offline-use) command to download images first, before running the pipeline. Setting the [`NXF_SINGULARITY_CACHEDIR` or `singularity.cacheDir`](https://www.nextflow.io/docs/latest/singularity.html?#singularity-docker-hub) Nextflow options enables you to store and re-use the images from a central location for future pipeline runs.
   > - If you are using `conda`, it is highly recommended to use the [`NXF_CONDA_CACHEDIR` or `conda.cacheDir`](https://www.nextflow.io/docs/latest/conda.html) settings to store the environments in a central location for future pipeline runs.

## Documentation

Currently this README serves as the only documentation for this workflow. More detailed documentation is under development. If you have specific questions, open a issue or connect at [mzanwar@sfu.ca](mailto:mzanwar@sfu.ca).

<!-- The nf-core/nfvibrio pipeline comes with documentation about the pipeline [usage](https://nf-co.re/nfvibrio/usage), [parameters](https://nf-co.re/nfvibrio/parameters) and [output](https://nf-co.re/nfvibrio/output).-->

## Credits

nf-core/nfvibrio was originally written by Muhammad Zohaib Anwar under the supervision of William Hsiao at the [Centre for Infectious Disease Genomics and One Health](http://www.cidgoh.ca/) at [Simon Fraser University](http://www.sfu.ca/).

<!--We thank the following people for their extensive assistance in the development of this pipeline:-->

<!-- TODO nf-core: If applicable, make list of people who have also contributed -->

## Contributions and Support

If you would like to contribute to this pipeline, please see the [contributing guidelines](.github/CONTRIBUTING.md).

<!-- For further information or help, don't hesitate to get in touch on the [Slack `#nfvibrio` channel](https://nfcore.slack.com/channels/nfvibrio) (you can join with [this invite](https://nf-co.re/join/slack)).-->

## Citations

<!-- TODO nf-core: Add citation for pipeline after first release. Uncomment lines below and update Zenodo doi and badge at the top of this file. -->
<!-- If you use  nf-core/nfvibrio for your analysis, please cite it using the following doi: [10.5281/zenodo.XXXXXX](https://doi.org/10.5281/zenodo.XXXXXX) -->

<!-- TODO nf-core: Add bibliography of tools and data used in your pipeline -->

An extensive list of references for the tools used by the pipeline can be found in the [`CITATIONS.md`](CITATIONS.md) file.

You can cite the `nf-core` publication as follows:

> **The nf-core framework for community-curated bioinformatics pipelines.**
>
> Philip Ewels, Alexander Peltzer, Sven Fillinger, Harshil Patel, Johannes Alneberg, Andreas Wilm, Maxime Ulysse Garcia, Paolo Di Tommaso & Sven Nahnsen.
>
> _Nat Biotechnol._ 2020 Feb 13. doi: [10.1038/s41587-020-0439-x](https://dx.doi.org/10.1038/s41587-020-0439-x).
