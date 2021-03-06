
# Please Note: Transition to Strelka2

There is no support or ongoing development for the Isaac Variant Caller package,
please instead see our ongoing updates to germline-calling in Strelka2, featuring **substantially
improved accuracy**, **GPLv3 source licensing**, **multi-sample support** and more:

https://github.com/Illumina/strelka

<br>

----

<br>

## Isaac Variant Caller -- 1.0.7

Christopher Saunders (csaunders@illumina.com)

Isaac Variant Caller (IVC) is an analysis package designed to detect
SNVs and small indels from the aligned sequencing reads of a single
diploid sample.

For additional information, please see:

https://github.com/sequencing


### WORKFLOW SUMMARY:

The workflow requires aligned sequencing reads in the BAM file
format. These reads must be sorted and indexed. PCR duplicate removal
is recommended. IVC does its own read realignment around indels so it
is not necessary to pre-process the data with any such step.

The output of the workflow is a single Genome VCF (gVCF) file
describing all sites and indels in the genome. gVCF is a set of
conventions for output in the VCF 4.1 format to represent all sites in
the genome. More information and gVCF processing utilities are
maintained at the following site:

https://sites.google.com/site/gvcftools/

For applications where only the variant calls are required, it is
recommended that only the passed variants (or a subset thereof) are
used. One way these can be extracted from the output is with the
'extract_variants' tool in the gvcftools package:

```bash
gzip -dc my_sample.genome.vcf.gz |\
extract_variants |\
awk '/^#/ || $7 == "PASS"' >\
all_passed_variants.vcf
```


### INSTALLATION:

The IVC workflow is designed to run on *nux-like platforms. It has
been specifically tested on the following distributions:

* Amazon Linux AMI 2011.09
* Centos 5.3
* Centos 6.2
* Mac OS X 10.5
* Ubuntu 11.10
* Ubuntu 12.04 LTS


For Centos/Amazon Linux and other rpm-based distributions, the
following packages (and their dependencies) are known requirements on
top of the base Amazon Linux AMI 2011.09:

* make
* gcc
* gcc-c++
* zlib
* zlib-devel


For Ubuntu and other Debian package distributions, the following
packages (and their dependencies) are known requirements on top of the
base Ubuntu 11.10 distribution:

* g++
* zlib1g-dev


Other compilation and runtime prerequisites are included in the
distribution tarball. These included prerequisites are:

* boost 1.44 (modified to include only a subset of the library)
* samtools 0.1.18 (modified to remove the 'curses' requirement)
* bgzf_extras 0.1
* codemin 1.0.2
* tabix 0.2.5 (makefile modified to reorder libz library to last
               place, as required for Ubuntu 12 build)


The Makefile included in the IVC workflow tarball will compile both
the included prerequisite packages and the variant caller itself.  To
do this, go to the root of the isaac_variant_caller_workflow directory
tree and run `./configure --prefix=$INSTALL_DIR` followed by make. 

Example build:
```bash
tar -xzf isaac_variant_caller_workflow.tar.gz
cd isaac_variant_caller_workflow
./configure --prefix=$INSTALLATION_DIR
make
```

If the installation succeeds, all files will be installed to
`$INSTALLATION_DIR`

The final installation step is to run the workflow on the demostration
data included in the distribution. Running the demo can be used to
verify that the workflow completes and produces the expected
answer. To do this, first complete the installation steps above. Then,
given a IVC installation path of `$IVC_INSTALL_DIR`, execute the
following script:

```
$IVC_INSTALL_DIR/bin/demo/run_demo.bash
```

The demo will create a 'ivcDemoAnalysis' directory under the current
working directory, then run an analysis for a very small genomic
region from a hapmap sample. It should complete in less than a minute.
If the demo script succeeds, you should see the following lines as the
final output:

```
**** No differences between expected and computed results.


**** Demo/verification successfully completed
```



### CONFIGURATION & ANALYSIS:

An IVC run is accomplished in two steps: (1) configuration and (2)
analysis.

The configuration is a fast step designed to setup the run and perform
some preliminary validation (eg. check that the chromosome names match
in the BAM header and reference genome). The configuration will create
a makefile to control the analysis and a directory structure to hold
all intermediate and final results. All configuration output is
written to the analysis directory. This step requires a configuration
initialization file containing IVC's default parameters. Template
configuration files can be found in the etc/ subdirectory under
the workflow installation root (see example run setup below).

Following configuration, the makefile can be used to run the analysis
itself using make, qmake, or a similar tool.

Example run:
```bash
# example location where strelka was installed:
#
IVC_INSTALL_DIR=/opt/isaac_variant_caller_workflow

# example location where analysis will be run:
#
WORK_DIR=/data/myWork

# Step 1. Move to working directory:
#
cd $WORK_DIR

# Step 2. Copy configuration ini file from default template set to a local copy,
#         possibly edit settings in local copy of file. In this example the
#         default configuration for whole genome sequencing is used. A
#         second default configuration exists for exome/targeted sequencing.
#
cp $IVC_INSTALL_DIR/etc/ivc_config_default_wgs.ini config.ini

# Step 3. Configure:
#
$IVC_INSTALL_DIR/bin/configureWorkflow.pl --bam=/data/input.bam --ref=/data/reference/hg19.fa --config=config.ini --output-dir=./myAnalysis

# Step 4. Run Analysis
#         This example is run using 8 cores on the local host:
#
cd ./myAnalysis
make -j 8
```


### CONFIGURATION DETAILS:

IVC includes a second default configuration file for exome/targeted sequencing -- the
only difference to WGS calling is that IVC's high depth filters are turned off for this
case. IVC high depth filtration is designed to remove pericentromeric reference
compression artifacts in WGS runs, but is not applicable for targeted sequencing. 

To use the exome/targeted configuration replace step 2 above with:

```
cp $IVC_INSTALL_DIR/etc/ivc_config_default_wes.ini config.ini
```
