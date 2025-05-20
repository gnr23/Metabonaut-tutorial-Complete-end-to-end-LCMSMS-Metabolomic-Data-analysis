In this tutorial, I am learning by using an end-to-end workflow for LC-MS-based untargeted metabolomics experiments, conducted entirely within R using packages from the Bioconductor project or base R functionality.


step 1 Data import

We extract our dataset from the MetaboLigths database and load it as an `MsExperiment` object. 

![img1](%5Cimages)

step 2 Data organization We next configure the parallel processing setup. Raw `.mzML` files contain millions of individual data points (mass-to-charge ratios, intensities, and retention times). processing them one by one on a single CPU core, can take hours. This code splits the workload so that **2 files are processed at the exact same time** using 2 separate cores of the processor.
img2

We are going to extract the metadata, rename the columns to something clean (like Sample, Type, Phenotype, Age), and save it back into our lcms1 object.

we can also rename the columns, add "Qc" for the qc samples , and injection indeces and print the updated table

img4

also by pre-defining a color mapping, we ensure that every single plot you make from this point forward (PCA plots, boxplots, chromatograms) 
will have the same consistent color key using the RColorBrewer package:

img 5

The MS data of this experiment is stored as a Spectra object (from the Spectra Bioconductor package) within 
the MsExperiment object and can be accessed using spectra() function

img6

spectraVariables: msLevel, rtime (retention time), precursorMz, and intensity. These are thmass spec data.

Total spectra:  the total count. This number represents every individual "scan" the mass spectrometer took during the entire run.

step3 Data visualization and general quality assessment. Effective visualization is paramount for inspecting and assessing the quality of MS data

BPC (Base Peak Chromatogram): We need the consistency of liquid chromatography. If the peaks in BPC shift wildly between samples, the retention times aren't repeatable 
and the alignment will fail.

TIC (Total Ion Chromatogram): Total sensitivity?. If one sample's TIC is significantly lower than the others, mass spectrometer might have been "dirty" or
 lost sensitivity during that specific run.

chromatogram function = The BPC can be extracted 

aggregationFun = "max", maximum signal per spectrum. 
aggregationFun = "sum",  all intensities of a spectrum, thereby creating a TIC.

img7 BPC of all samples colored by phenotype.
 
we will filter the data removing that part after 200 s as well as the first 10 seconds measured in the LC run.
and plot the BPC for each sample colored by phenotype, providing insights on the signal measured along the retention times of each sample

img8
  a BPC condenses the three-dimensional LC-MS data (m/z by retention time by intensity) into two dimensions (retention time by intensity).

Internal standards EIC

Reference points for all of our samples. By restricting the MS data to intensities within a restricted, m/z range and a selected retention time window, EICs are expected to contain only signal from a single type of ion

Here only two isotopically labelled IS are used, methionine and cysteine

img9 importing them  If wetell the software to look for mz = 249.0453, it looks for that exact number. Because of electronic noise and instrument calibration, the machine might record it as 249.0454 or 249.0452. If wedon't provide a window (mzmin to mzmax), the result returns "empty." This is why real-world metabolomics code always includes these small +/- tolerances.

then we plot the EIC for both

img10

we immediately notice retention time shift as well as differences in intensities between samples and QC
