# Prepare sample sheet

In the RAPID spreadsheet, prepare the RAPID ids and taxon ids to make sure the have nothing except alphanumeric characters and underscores.

Make a .csv with the RAPID id in the first column and taxon id in the second column. For example:

```
BYU_105603_P030_WA01,GD_5101_Cordulegastridae_Anotogaster_sieboldii
BYU_105603_P030_WA02,GD_5102_Cordulegastridae_Zoraena_bilineata
BYU_105603_P030_WA03,GD_5103_Cordulegastridae_Cordulegaster_boltonii
```

Navigate to the RAPID delivery folder. Use this command to find R1 and R2 .fastq files for each sample. The input csv you made above is named `rapid_spreadsheet_input.csv` and the output csv will be named `samplesheet.csv`.

```
echo 'sample,fastq_1,fastq_2' > samplesheet.csv
while read line; do
    rapid_id=$(echo $line | cut -d ',' -f 1)
    taxon=$(echo $line | cut -d ',' -f 2)
    R1=$(find . -iname "*${rapid_id}*R1*")
    R2=$(find . -iname "*${rapid_id}*R2*")
    echo "${taxon},${R1},${R2}" >> samplesheet.csv
done < rapid_spreadsheet_input.csv
```

# Download the pipeline, install dependencies

Download the `nf-core-targetassembly` pipeline from GitHub:

```
git clone https://github.com/paytoncarter14/nf-core-targetassembly.git
```

The only dependencies you need to install are Nextflow and Apptainer. The rest of the software will be automatically installed by the pipeline using Apptainer containers.

To use Conda to create an environment with Nextflow and Apptainer:

```
conda create -n nextflow-24.10.4 -c conda-forge -c bioconda nextflow=24.10.4 apptainer=1.3.6
conda activate nextflow-24.10.4
```

# Run assembly

Run the pipeline. The finished assembly .fasta files will be in `output/(probe,full)_orthologs`. Everything else (e.g. fastp reports) will be in the `intermediates` folder.

This example uses the 20kb probe set. Change the `--probes` file if you're assembling a 500kb plate.

```
nextflow run nf-core-targetassembly/main.nf \
    --reference /nobackup/archive/grp/grp_geode/scripts_and_files/assembly/tanypteryx.fasta \
    --probes /nobackup/archive/grp/grp_geode/scripts_and_files/assembly/R1_tanypteryx1306_20kbp_R.fasta \
    --input samplesheet.csv \
    --outdir output \
    -process.executor slurm \
    -profile apptainer \
    -resume
```