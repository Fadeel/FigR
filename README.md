# Functional Inference of Gene Regulation (FigR)
------------------------------
FigR code was updated to do the following,
1) accept any collection of motifs (formated as PFMatrixList, from TFBSTools package)
2) one TF can have multiple motifs. For each DORC, pick TF motif with the highest enrichment to be associated with this TF
------------------------------

FigR is a computational framework for supporting the integration of single cell chromatin accessibility and gene expression data to infer transcriptional regulators of target genes


<p align="center">
<img src="man/figures/FigR.PNG" height="50%" width="50%">
</p>

FigR uses independently or concomitantly-profiled single-cell ATAC-seq (scATAC-seq) and scRNA-seq, to 
i) computationally-pair scATAC-seq and scRNA-seq datasets (if needed), ii) infers cis-regulatory interactions, and iii) defines a TF-gene gene regulatory network (GRN)


# Installation

`devtools::install_github("buenrostrolab/FigR")`

# Quickstart

### DORC calling

If you have a `SummarizedExperiment` object of the scATAC-seq reads in peaks counts (peaks x cells), as well as a `sparseMatrix` object of the scRNA-seq gene counts (genes x cells) that are paired (either computationally or true multi-modal), you can determine *cis*-regulatory associations (gene-peak correlations) as follows

```
# Run using multiple cores if parallel support
cisCor <- runGenePeakcorr(ATAC.se = ATAC.SE,
                           RNAmat = rnaMat,
                           genome = "hg19", # Also supports mm10 and hg38
                           nCores = 4, 
                           p.cut=NULL)
                    
# Filter peak-gene correlations by p-value                    
cisCor.filt <- cisCor %>% filter(pvalZ <= 0.05)

# Determine DORC genes
dorcGenes <- cisCor.filt %>% dorcJPlot(cutoff=7, # Default
                                       returnGeneList = TRUE)

# Get DORC scores
dorcMat <- getDORCScores(ATAC.SE,dorcTab=cisCor.filt,geneList=dorcGenes,nCores=4)

# Smooth DORC scores (using cell KNNs)
dorcMat.smooth <- smoothScoresNN(NNmat=cellKNN.mat,mat=dorcMat,nCores=4)

```

*Cis*-regulatory correlation analysis framework to identify gene-peak (chromatin accessibility peak) significant associations and deduce key genes that are domains of such regulatory activity (DORCs)

### FigR 

Once you have your DORC definitions, you can use this, along with the paired RNA data, to run the core FigR function. This returns a table of regulations cores between each DORC gene and a TF in the reference database, which we can filter on and visualize (see section below)

```
# Run FigR
fig.d <- runFigRGRN(ATAC.se=,ATAC.SE,
                    rnaMat=rnaMat.smooth, # Smoothed RNA matrix using paired cell kNNs
                    dorcMat=dorcMat.smooth,
                    dorcTab=cisCor.filt,
                    genome="hg19",
                    dorcGenes=dorcGenes,
                    nCores=4)

```

### Visualizing FigR results

```

# Visualize all TF-DORC regulation scores (Scatter plot)
require(ggplot2)
require(ggrastr)
require(BuenColors) # https://github.com/caleblareau/BuenColors

fig.d %>% ggplot(aes(Corr.log10P,Enrichment.log10P,color=Score)) +
          geom_point_rast(size=0.01,shape=16) + 
          theme_classic() +
          scale_color_gradientn(colours = jdb_palette("solar_extra"),limits=c(-4,4),oob = scales::squish)


```

### What reference genomes does FigR currently support?

FigR currently has built in support for human reference genomes hg19 and hg38, and mouse reference genome mm10. We will soon provide support for user-defined input genome objects (has to be supported by the `BSgenome` suite of packages) as well as objects with TSS coordinates per gene (of class `GRanges`). Users are responsible for ensuring the right reference genome object is paired with the appropriate TSS gene object, if specified 

# About

We are actively updating this package to help with user requests!

Please visit our [documentation website](https://buenrostrolab.github.io/FigR/) for more details on examples and functions that the FigR package supports.

You may also directly check out our walk-throughs on applying the FigR framework to multi-modal and independently derived scATAC-seq / scRNA-seq data by clicking on the links below:

<p align="left">
<a href="https://htmlpreview.github.io/?https://raw.githubusercontent.com/buenrostrolab/FigR/master/vignettes/web_only/FigR_shareseq_tutorial.html"><img src="man/figures/skinv2.png"  title="FigR on SHARE-seq mouse skin data" height="30%" width="35%"></a>

</p>

<p align="center">
<a href="https://htmlpreview.github.io/?https://raw.githubusercontent.com/buenrostrolab/FigR/master/vignettes/web_only/FigR_stim_tutorial.html"><img src="man/figures/PBMCs.png"  title="FigR on independently assayed PBMC data" height="30%" width="35%"></a>
</p>

# Reference / Citation

See our [published manuscript](https://www.sciencedirect.com/science/article/pii/S2666979X22001082) detailing  the use of the FigR framework for gene regulatory network inference using scATAC and scRNA-seq integration
