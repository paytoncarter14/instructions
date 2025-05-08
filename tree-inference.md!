# Prepare fasta directory

Prepare `taxa_list.txt` and `exclude_list.txt` with sample names. One taxa per line. Searches are case-sensitive and based on filenames, so make sure to enclose taxa in underscores to make sure only the taxa you want are included. For example, 'Coenagrion' will match 'Coenagrion' and 'Coenagrionidae', while '\_Coenagrion\_' will only match the genus 'Coenagrion'.

Remove `/20kb` if you want both 20kb and 500kb samples. Replace `probe` with `full` if you want full assemblies.

```
mkdir fasta
while read line; do 
    find /grphome/grp_geode/assemblies_cleaned/probe/20kb -name "*${line}*" -exec ln -s {} fasta \;
done < taxa_list.txt
while read line; do
    find fasta -name "*${line}*" -delete
done < exclude_list.txt
```

# Download pipeline

```
git clone https://github.com/paytoncarter14/nf-core-treeinference
```

# Run pipeline

```
nextflow run nf-core-treeinference \
    -profile apptainer \
    -process.executor slurm \
    -resume \
    --input fasta \
    --outdir output
```
