# make-tree-inference-fasta-from-sqlite.py

This script generates locus `.fasta` files from the GEODE sqlite database, which can then be used as input for the nf-core-treeinference pipeline. You can run it with, for example:

```bash
python make-tree-inference-fasta-from-sqlite.py \
	--taxonomy taxonomy.csv \
	--outdir mysamples \
	--loci 20kb \
	--region probe \
	--min-avg-kmer-coverage 10 \
	--min-locus-kmer-coverage 20 \
	--min-num-loci-20kb 50 \
	--min-num-loci-500kb 50
```

## loci and region

For the `--loci` parameter, you can specify `all`, `20kb`, or `500kb` to select those loci sets. You can also provide a newline separated file of specific loci to use, such as:

```
L1004__tanypteryx_R
L995__tanypteryx_R
L2001__tanypteryx_R
```

For the `--region` parameter, you can specify either `probe` or `full` to use either just the probe region or the full scaffold (probe + flanks).

## kmer coverage

When these samples were assembled with SPAdes, we collected the kmer coverage statistic for each locus. This is the average number of times each kmer was found in this locus. Generally, a good locus should have at least 10x kmer coverage. The capture probes differ in their capture efficiency, so different loci in the same sample can have very different levels of kmer coverage. This script allows you to filter both individual loci and entire samples by the kmer coverage values. **By default, no filtering is applied.** You will probably want to set some values for the following parameters:

- `--min-avg-kmer-coverage`: The minimum average SPAdes kmer coverage for the entire sample. If a sample is lower than this, the entire sample is excluded.
- `--min-locus-kmer-coverage`: The minimum SPAdes kmer coverage for each locus. If a locus is lower than this, the individual locus is excluded for this sample.
- `--min-num-loci-20kb`: The minimum number of loci after the kmer coverage filter for 20kb samples. If there are fewer loci than this number, the entire sample is excluded.
- `--min-num-loci-500kb`: The same as above, but for 500kb samples.

You will probably want the same `--min-num-loci` value for both 20kb and 500kb, unless you plan to use all loci for both 20kb and 500kb samples.

For example, the following parameters will:

```bash
--loci 20kb \
--min-avg-kmer-coverage 10 \
--min-locus-kmer-coverage 20 \
--min-num-loci-20kb 50 \
--min-num-loci-500kb 50
```

- First pull the 20kb loci for all selected samples.
- Each sample has a kmer coverage value that is averaged across all loci. In this case, any sample with an average of less than 10 is filtered.
- Next, any individual loci with a kmer coverage less than 20 is filtered.
- Finally, the remaining loci are counted for each sample. For both 20kb and 500kb samples, any with fewer than 50 loci remaining are filtered.

The script will print how many samples and sequences remain at each filtering step. You can also find the `selected-samples.csv` and `selected-sequences.csv` files in the output directory, which will give you statistics for each sample.

## taxonomy

The `--taxonomy` argument should be a .csv file with the header `family,genus,species,rapid_id`. When this file is parsed:

- Each row will be matched as specifically as possible. For example, if a rapid\_id is provided, all other fields will be ignored. If a genus is provided, the family will be ignored.
- If any field begins with !, the search will exclude any matching samples for that row.
- If a species is provided, a genus must also be provided.
- You can use the `%` character as a wildcard.

For example:

    family,genus,species,rapid_id
    Coenagrionidae,,,
    ,!Ischnura,elegans,
    ,,,105603_P004_WC11
    ,,,!105602_P002_%

will include all samples from the family `Coenagrionidae`, except for those from the species `Ischnura elegans`. It will also specifically include the sample `105603_P004_WC11` and exclude any samples starting with `105602_P002_`.

## nf-core-treeinference

The output directory can be directly given to nf-core-treeinference as the input directory. You will need to specify `input_format = 'locus'` in the parameters, since there will be one `.fasta` file per locus instead of the default one per sample. You can do this, for example, through the command line:

```bash
nextflow run /grphome/grp_geode/payton/git/nf-core-treeinference \
	--input mysamples \
	--outdir output \
	--input_format locus \
	-profile apptainer \
	-resume
```

or with a `nextflow.config` file in the run directory:

```groovy
params {
	input = 'mysamples'
	outdir = 'output'
	input_format = 'locus'
}
resume = true
```
