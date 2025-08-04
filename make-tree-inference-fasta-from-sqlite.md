# make-tree-inference-fasta-from-sqlite.py

Use this script to generate locus `.fasta` files from the GEODE sqlite database for the nf-core-treeinference pipeline.

You can run it with, for example:

```bash
python make-tree-inference-fasta-from-sqlite.py \
	--taxonomy taxonomy.csv \
	--outdir mysamples \
	--loci 20kb \
	--region probe
```

This will read taxonomy information from the `taxonomy.csv` file. It will create one `.fasta` file per 20kb locus with probe sequences in the `mysamples` directory.  You can override the default values for:

- `--min-avg-kmer-coverage`: default = 30. The minimum average SPAdes kmer coverage for the entire sample. If a sample is lower than this, the entire sample is excluded.
- `--min-locus-kmer-coverage`: default = 30. The minimum SPAdes kmer coverage for each locus. If a locus is lower than this, the individual locus is excluded for this sample.
- `--min-num-loci-20kb`: default = 20. The minimum number of loci after the kmer coverage filter for 20kb samples. If there are fewer loci than this number, the entire sample is excluded.
- `--min-num-loci-500kb`: default = 20. The same as above, but for 500kb samples.

The `--taxonomy` argument should be a .csv file with the header `family,genus,species,rapid_id`. Each row will be matched as specifically as possible. For example, if a rapid\_id is provided, all other fields will be ignored. If a genus is provided, the family will be ignored. If any field begins with !, the search will exclude any matching samples for that row. If a species is provided, a genus must also be provided.

For example:

    family,genus,species,rapid_id
    Coenagrionidae,,,
    ,!Ischnura,elegans,
    ,,,105603_P004_WC11
    ,,,!105602_P002_WA05

Will include all samples from the family Coenagrionidae, except for those from the species `Ischnura elegans`. It will also specifically include the sample `105603_P004_WC11` and exclude the sample `105602_P002_WA05`.

The output directory can be directly given to nf-core-treeinference as the input directory. You will need to specify `input_format = 'locus'` in the parameters, since there will be one `.fasta` file per locus instead of the default one per sample. You can do this, for example, through the command line:

```bash
nextflow run /grphome/grp_geode/payton/git/nf-core-treeinference \
	--input mysamples \
	--outdir output \
	--input_format locus \
	-profile apptainer \
	-resume
```
