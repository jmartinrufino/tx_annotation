# Transcript expression-aware annotation 

Welcome to the repository for the "Transcript expression-aware annotation improves rare variant discovery and interpretation" manuscript. Here we'll outline how to get transcript expression values for your variant file and isoform expression expression matrix of interest, and outline the commands and code to recreate analyses in the pre-print. 

## Applying transcript expression aware annotation to your own dataset

You will need 
  1) A variant file that has columns for chrom, pos, ref and alt
  2) An isoform expression matrix 
  3) The ability to use Hail locally or on a cloud platform 
  
You can have additional columns in your variant file, which will be maintained, and only new columns of transcript-expression annotation will be added. Your isoform expression matrix must start with two columns : 1. transcript id and 2. gene_id. The remaining columns can be any samples or tissues. If you have biological replicates, they should be numbered with a '.' delimiter (e.g. MuscleSkeletal.1, MuscleSkeletal.2, MuscleSkeletal.3). 
 
Instructions to set up Hail are laid out in the [Hail docs](https://hail.is/docs/0.2/)

If you're unable to set up Hail in your local environment, we have also release the pext values for every possible SNV in the genome: gs://gnomad-public/papers/2019-tx-annotation/pre_computed/all.possible.snvs.tx_annotated.021819.tsv.bgz

More information about this files format is below.

Please be aware that while we don't expect any issues, the files may be iterated upon until publication of the manuscript! 

We will walk through an example of annotating *de novo* variants in autism and developmental delay / intellectual disability with the GTEx v7 dataset 

###### 1) Prepare the variant file 
The variant file is available at : gs://gnomad-public/papers/2019-tx-annotation/data/asd_ddid_de_novos.txt

This is what the first line of the file looks like : 
      DataSet CHROM POSITION REF ALT Child_ID Child_Sex GENE_NAME VEP_functional_class_canonical MPC loftee   group
1 ASC_v15_VCF     1 94049574   C   A 13069.s1      Male     BCAR3           splice_donor_variant  NA     HC Control

Again, we will only use the chrom, pos, ref, alt columns, and will add additional columns. 

In order to add pext values, you must annotate with VEP. This is how to import the file into Hail, define the variant field, vep, and write the MT. 

1 - Import file as a table 
```
rt = hl.import_table("gs://gnomad-public/papers/2019-tx-annotation/data/asd_ddid_de_novos.txt")
```
2 - Define the variant in terms of chrom:pos:ref:alt and have Hail parse it, which will create a locus and alleles column
```
rt = rt.annotate(variant=rt.CHROM + ':' + rt.POSITION + ":" + rt.REF + ":" + rt.ALT)
rt = rt.annotate(** hl.parse_variant(rt.variant))
rt = rt.key_by(rt.locus, rt.alleles)
```
3 - Make a MT from table, and repartition for speed (rule of thumb is ~2k variants per partition)
```
mt = hl.MatrixTable.from_rows_table(rt)
mt = mt.repartition(10)
```
4 - VEP and write out the MT
```
annotated_mt = hl.vep(mt, vep_config)
annotated_mt.write("gs://gnomad-public/papers/2019-tx-annotation/results/de_novo_variants/asd_ddid_de_novos.vepped.021819.mt")
```

###### 2) Prepare the isoform expression file 

We'll use the GTEx v7 isoform expression file. Here is what the header of the file looks like 


We've replaced the sample names with unique tissue names, so that samples with the same tissue are labelled as WholeBlood.1, WholeBlood.2, WholeBlood.3 etc:



###### 3) Add pext values

## Analyses in manuscript 

###### Conservation and pext
d
