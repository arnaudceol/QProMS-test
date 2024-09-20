## QProMS - Quantitative PROteomics Made Simple

![Logo](https://github.com/FabioBedin/QProMS_App/blob/main/app/static/QProMS-logo.png)


QProMS is a shiny app that enables easy but powerful and reproducible analyses for label-free proteomics data. It works out of the box with  major data-dependent and data-independent search engine results (MaxQuant, FragPipe, Spectronaut, DIA-NN, AlphaPept) as well as custom result tables. It can produce publication-quality figures and export HTML reports and parameter files for sharing and reproducing results. It can handle multiple simultaneous comparisons between different experimental conditions.

### Installation or running online

Open the app [here](https://bioserver.ieo.it/shiny/app/qproms). The app can also be run locally. Open Rstudio (make sure you have RTools installed along with R>4.3) and open this project from a local download or by loading it from version control. Then type:


    renv::restore(rebuild=TRUE)


To install all dependencies. You can then load the app by


    shiny::runApp()

    
and heading to the address in your browser.


### Getting started

The app guides you through a typical analysis workflow for proteomics:

1. Data upload and experimental design annotation
2. Quality control and handling missing values
3. Statistical analysis (clustering, volcano and more)
4. Network and functional analysis
5. Exporting results.

The app works on any proteomics search engine result table, provided it has a column with Protein IDs/Gene names and intensity/LFQ/iBAQ or similar columns corresponding to each experiment. It requires a quantitative experiment performed in at least triplicates in 2 conditions or more to compare.

In all sections, the options pre-selected are defaults that should apply to most scenarios including global proteome profiling, AP-MS and proximity labelling MS. Any change in the options can be effected by clicking "update" in each individual page, and those changes will be propagated throughout the app. For example, if the user changes imputation strategy, even after cdisplaying a volcano plot, the volcano plot will be automatically be updated with the newly selected imputation.

In the global settings in the top-right wheel, the user may change palettes for the plots, plot text size, and whether plots will be downloaded in svg or png format.

### Upload page

Upload search engine results. Files should have a column for gene IDs, and columns for quantitation of each protein in the respective conditions. Several search engines are supported by default. Other tables can also be uploaded and then fields defined manually by selecting the appropriate columns. File names are arbitrary, but the files to upload for each search engine are


| Software    | File to upload (default name)  | Comment                                                                                                                                                            | 
|-------------|--------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------| 
| MaxQuant    | ProteinGroups.txt              | Must contain "Gene Names" column. QProMS includes automatic parsing of reverse, contaminants and only identified by site and handling of ambiguous protein groups. |
| FragPipe    | combined_protein.tsv           | QProMS has automatic annotation of contaminants included by Philosopher/FragPipe.  |
| Spectronaut | Report.tsv                     |  | 
| Dia-NN      | report.unique_genes_matrix.txt | |  
| AlphaPept   | results_proteins.csv           | |  

Select the organism of your data and and click on "start". If your organism is not available or gene names are missing, the app will still work, but some functions such as GO enrichment will be missing. The app will automatically detect which result table you have if automatic detection fails because of a non-standard table. You may then select which columns of your table represent gene names and which ones are intensities from the left side menu.

For AlphaPept, since the program does not parse gene names automatically, Network, ORA and GSEA are disabled by default unless the user has parsed gene names from Protein ID. The user may then manually select which column correspond to gene names in his or her dataframe.

If you have worked with QProMS before and are trying to reproduce an analysis or an earlier session, you can tick on "load parameters" and upload the QProMS .yml file generated by ["Generate parameters"](https://github.com/FabioBedin/QProMS?tab=readme-ov-file#report-generation-and-reproducing-the-analysis) together with your protein table.

All intensities are automatically log2 transformed in the upload phase unless specified otherwise. 

After clicking on "start" you can edit your experimental design in the table that appears below. It is modelled after the MaxQuant and FragPipe experimental design tables.


| Column name | Meaning                                                                                                                                                                   |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| keep        | Whether to keep or discard this column for the QProMS analysis                                                                                                            |
| condition   | The experimental condition (e.g. control/tumor, time point etc). Can be letters and/or numbers. All replicates of one condition must have the same condition name         |
| key         | Unique name for the column that is only used by QProMS (not editable)                                                                                                     | 

Set your experimental conditions, click on "make design table" and then "start".


### Preprocessing


The "Preprocessing" page displays protein counts, upset plot, intensity distributions and the distribution of coefficients of variation (CVs). 

The left panel for "parameters" can be expanded to reveal some settings that can be changed. All settings will propagate through later steps of the analysis and saved in the report.

The user can tweak filtering of proteins on valid values based on how often they are observed in at least one experimmental condition (group), in all groups, or in total.

Normalization across samples using variance stabilizing normalization (VSN) can be toggled on here.

Other options available here are dependent on the search engine used in the upload page: 

- For MaxQuant, the app automatically filters proteins to those with minimum 2 peptides detected (editable by the user), and removes contaminants, reverse and only identified by site (also editable by the user).

- For FragPipe, contaminants are automatically removed based on the [CrapOme list](https://reprint-apms.org/?q=about) utilized by FragPipe and Philosopher during database construction.

#### Imputation

From within the preprocessing page, the algorithm for imputation may be selected. There are 2 modes: mixed imputation, introduced in the [QProMS publication](XXX), and the Perseus algorithm, adapted from [here](http://www.coxdocs.org/doku.php?id=perseus:user:activities:matrixprocessing:imputation:replacemissingfromgaussian). In the Perseus algorithm, missing values are replaced by sampling from a down-shifted gaussian distribution. The user may also opt for no imputation.

The mean of the imputed distribution can be positioned relative to the mean experimental intensity in terms of standard deviations with the "down shift" parameter. The width of the imputed distribution can be adjusted by the "scale" parameter.

Mixed imputation recognises two types of missing data: missing (MAR) at random and missing not at random (MNAR). Missing at random values (i.e. those missing from a single replica in a condition) are imputed with the mean of the value of that protein in the other replicas. Missing not at random (completely missing from a particular experimental condition) are imputed with the Perseus-style algorithm.


The missing data tab presents an UpSet plot so that the user may verify which experiments are mostly affected by missing values.

In the imputed tab the user can visualize the distribution of missing values and the effects imputation. The contribution from imputation can be highlighted in the "Imputed" plot. The distribution per experiment can be visualized by scrolling through the plots in the trelliscope panel.


In the last panel, the table post data filtering and imputation (if imputation is used) is available for export and interactive searching.


### Principal component analysis (PCA)

The page displays a principal component analysis in 2d and 3d. The top of the page highlights the variance explained by each principal component. These plots are useful as quality control to ensure that replicas cluster with each other and separate across different experimental conditions (i.e. that most of the variance is contributed by changing of experimental condition).

### Correlation

Correlation matrix for the data. Scatterplots for individual experiments are available in the "Scatter plots" panel. Individual proteins may be highlighted by selecting them in the table.

Users can select the type of correlation analysis by clicking on the parameters tab to set Pearson (default), Spearmann or Kendall correlation.


### Rank

The rank page visualizes the abundance rank of each protein. In the Table panel, the user may search and highlight one or more protein of interest which are then displayed in the main plot.

The "merge condition" toggle refers to whether abundances are plotted from individual experiments of from the mean of all experiments in one condition.


### Volcano

This page guides the user through differential analysis and generation of the volcano plot. 

In Inputs, the user selects which conditions to compare. Each comparison is written as "enriched_vs_control". Multiple comparisons may be selected, each of which will be displayed as its own volcano plot.

In "Parameters", the user selects the type of statistical test for generation of the p-value and the FDR correction (truncation).

- Welch's T-test (default): two-sample t-test that is more reliable when populations have unequal variances and different sample sizes.
- Student's T-test: two-sample t-test that assumes populations have equal variance.
- limma (linear models for microarray and RNA-Seq): Bayesian univariate test relying on row-wise linear models to estimate likelihood of differental expression. More information [here](https://academic.oup.com/nar/article/43/7/e47/2414268).

The "paired" option is off by default but can be toggled if a paired test is to be performed. This is desirable if the comparison is between two samples that are correlated.

"Fold change" is the minimum fold change (in log2 units) to consider as significant.

Alpha is the level of significance after correction for multiple testing (Truncation) is applied. Several corrections are available, with the default being Benjamini/Hochberg FDR correction.

### Heatmap
The heatmap page provides clustering analysis based on protein intensities. The user selects the number of clusters and the hierarchical clustering method from those available in R's hclust function.

The default, "complete", computes all pairwise dissimilarities between the elements in cluster A and the elements in cluster B, and considers the largest value (i.e., maximum value) of these dissimilarities as the distance between the two clusters. It tends to produce more compact clusters."Ward's method aims to minimize the total within-cluster variance. Both methods are commonly used in proteomics research.

The user may also select significance threshold (alpha) and FDR correction method.

The cluster profile panel provides profile plots for clusters with their average abundance and 95% confidence intervals across conditions. 

After selecting one or more proteins of interest in the table panel, the profile plots of individual proteins may be visualized in the protein profile panel.

### Network

The network page can be used to infer protein-protein interactions. The proteins to analyse can be selected from what is enriched in the volcano analysis, the top ranking proteins, or from specific clusters in the heatmap.

The view then produces a network of proteins where the edges are protein pairs annotated in [String](https://string-db.org) (or [CORUM](https://mips.helmholtz-muenchen.de/corum/), if added to the selection) and the edge size is the [String score](https://string-db.org/cgi/help?sessionId=b8E7e9gCqSJu).

This may be used as a starting point for further network analysis, for example in Cytoscape.

### Over-representation analysis (ORA)

This page performs over-representation analysis with Gene Ontology (GO) terms. It can be used to identify which biological processes, molecular functions or cellular components are enriched or depleted in the dataset. The user may select proteins from the volcano analysis, specific clusters from the heatmap, top ranking proteins or a manual selection.

If multiple comparisons are carried out, one may select multiple contrasts to generate multiple plots.

In "parameters", the user may define the "simplify threshold", the user may select how much grouping of GO terms is performed. At low values, for example, "40S ribosome", "80S ribosome" may be grouped into "ribosome" or even "translation". 

If "Bacgkground" is disabled (default), ORA is performed against the organsim's whole proteome. Enabling it will perform ORA against the entire list of proteins identified in the upload.

In visual parameters, the user may select whether the bar chart displays the fold enrichment, statistical significance (-log of p value or FDR-corrected p-value) or simple count of proteins.



### Geneset enrichment analysis (GSEA)

This page performs geneset enrichment analysis (GSEA) with Gene Ontology (GO) terms. It can be used to identify which biological processes, molecular functions or cellular components are enriched or depleted in the dataset. Unlike ORA, GSEA is based on the list of all proteins in one or more conditions and it then finds which GO terms are more likely to be among the most abundant, expressed as a normalized enriched score (NES). Thus, it assesses whether proteins associated with predefined gene sets (e.g., pathways or functional categories) are predominantly found at the top or bottom of the ranked list.

In "parameters", the user may define the "simplify threshold", the user may select how much grouping of GO terms is performed. At high values, for example, "40S ribosome", "80S ribosome" may be grouped into "ribosome" or even "translation". 

The user may define the level of significance (Alpha) and the FDR correction method. In visual parameters, the user may select whether the bar chart displays the normalized enriched score, statistical significance (-log of p value or FDR-corrected p-value) or the number of proteins making up each set.

**This is a bit more computationally intensive than other functions in QProMS, so the analysis may take a few minutes.**


### Report generation and reproducing the analysis

The app can generate an HTML report that may be shared with collaborators. Clicking on the top right download button enables report download or download of individual tables in excel or csv format as well as individual figures.

Finally, the entire analysis session may be downloaded here. The resulting file in .rds format can then be used down the road to reopen the session in the upload page.


### Advanced usage- custom contaminants lists

Users running QProMS locally may edit the contaminants list used by the app for non-MaxQuant searches by editing the contents of the file contaminants.R in apps/static with IDs matching those found in the results table.

go
