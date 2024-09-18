# Step 1: Convert VCF to PLINK format
PLINK can work with VCF files directly, but it's common to convert them into binary PLINK format (.bed, .bim, .fam).

bash
```
plink --vcf input_cases.vcf --make-bed --out vitiligo_cases
plink --vcf input_controls.vcf --make-bed --out healthy_controls
````
This will convert the VCF files to PLINK's binary format.

# Step 2: Merge case and control datasets
After conversion, you'll need to merge the two datasets. PLINK requires the files to have compatible formats.

bash
```
plink --bfile vitiligo_cases --bmerge healthy_controls.bed healthy_controls.bim healthy_controls.fam --make-bed --out merged_data
```
If there are mismatching SNPs between the datasets, PLINK will report errors. You can exclude the problematic SNPs and rerun the merging step.

# Step 3: Quality control (QC)
Before performing an association analysis, it's essential to clean your data through QC steps. Common checks include missingness, minor allele frequency (MAF), and Hardy-Weinberg equilibrium (HWE).

## Missing genotype rate:
bash
plink --bfile merged_data --geno 0.05 --make-bed --out qc1
This excludes SNPs with a missing genotype rate > 5%.

## Minor Allele Frequency (MAF):
bash
```
plink --bfile qc1 --maf 0.01 --make-bed --out qc2
```
This filters out SNPs with a MAF < 1%.

## Hardy-Weinberg equilibrium (HWE):
bash
```
plink --bfile qc2 --hwe 1e-6 --make-bed --out qc3
```
This excludes SNPs that deviate from HWE (in controls only).

## Sample missingness:
bash
```
plink --bfile qc3 --mind 0.05 --make-bed --out qc4
```
This removes samples with a high missing genotype rate (> 5%).

# Step 4: Principal Component Analysis (PCA)
It’s essential to account for population stratification. You can perform PCA to generate covariates for your analysis.

bash
```
plink --bfile qc4 --pca 10 --out pca_results
```

The pca_results.eigenvec file contains the principal components you can use as covariates in the association analysis.

# Step 5: Case-control association analysis
Now, you can run the association analysis using logistic regression.

bash
```
plink --bfile qc4 --logistic --covar pca_results.eigenvec --covar-number 1-10 --out association_results
```

This will perform a logistic regression with the first 10 principal components as covariates.

# Step 6: Visualize results (Manhattan plot and QQ plot)
After the association analysis, it's helpful to visualize the results. You can use the following R script to create Manhattan and QQ plots:

r
```
library(qqman)
```

## Load results
```
assoc <- read.table("association_results.assoc.logistic", header=TRUE)
```

## Manhattan plot

```
manhattan(assoc, chr="CHR", bp="BP", snp="SNP", p="P", main="Manhattan Plot", ylim=c(0, 10))
QQ Plot (R)
r
library(qqman)
```
## QQ plot
```
qq(assoc$P, main="QQ Plot of Association P-values")
```

