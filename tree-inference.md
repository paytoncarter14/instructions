# Prepare fasta directory

Prepare `taxa_list.txt` and `exclude_list.txt` with sample names. One taxa per line. Searches are case-sensitive and based on filenames, so if you're searching by family or genus, make sure to enclose taxa in underscores to make sure only the taxa you want are included. For example, 'Coenagrion' will match 'Coenagrion' and 'Coenagrionidae', while '\_Coenagrion\_' will only match the genus 'Coenagrion'. If you're searching by sample ID, put the underscore only at the end ('GEODE1234_').

Remove `/20kb` if you want both 20kb and 500kb samples. Replace `probe` with `full` if you want full assemblies.

The following command creates the `fasta` folder and symlinks the assemblies that match in `taxa_list.txt`.

```
mkdir fasta
while read line; do 
    find /grphome/grp_geode/assemblies_cleaned/probe/20kb -name "*${line}*" -exec ln -s {} fasta \;
done < taxa_list.txt
```

If you have taxa to exclude in `exclude_list.txt`, this will delete those symlinks.

```
while read line; do
    find fasta -name "*${line}*" -delete
done < exclude_list.txt
```

# Download pipeline and load Nextflow/Apptainer

:exclamation:***You only need to do the steps in this section once.***:exclamation:

This downloads the pipeline.

```
git clone https://github.com/paytoncarter14/nf-core-treeinference
```

This loads Apptainer from the supercomputer module library and saves it so you don't need to load it every time.

```
module load apptainer
module save
```

Download and install Nextflow by following the instructions at the top of this website (the box with three steps): <https://www.nextflow.io/>

# Run pipeline

This runs the pipeline with Apptainer, submits the jobs to the Slurm scheduler, and stores intermediate work files on the autodelete partition. Output files will be in the `output` folder.

```
nextflow run nf-core-treeinference \
    -profile apptainer \
    -process.executor slurm \
    -work-dir ~/nobackup/autodelete/work \
    -resume \
    --input fasta \
    --outdir output
```
