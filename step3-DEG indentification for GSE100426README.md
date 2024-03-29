## 
### ---------------
### Date: 2020-2-24 
### Step3:load GSE100906 expression file and filter,find DEG
### ---------------

# read data
a=read.table('GSE100426.txt',row.names= 1,  header = T ,sep = '\t') 

# check data
a[1:6,1:4] 

library(dplyr)
library(Seurat)
library(stringr)

# Create Seurat Object
GSE100426<- CreateSeuratObject(counts = a, project = "GSE100426", min.cells = 3, min.features = 200)
# filter cells and analysis LT-HSC only
GSE100426=subset(GSE100426,idents = "LT")

# filter data
age=matrix(data=NA, nrow = ncol(GSE100426), ncol = 1)
condition=matrix(data=NA, nrow = ncol(GSE100426), ncol = 1)
plate=matrix(data=NA, nrow = ncol(GSE100426), ncol = 1)
for(i in 1:ncol(GSE100426)){
  if(str_detect(colnames(GSE100426)[i],"young")) age[i,1]="Young"
  if(str_detect(colnames(GSE100426)[i],"old"))   age[i,1]="old"
  if(str_detect(colnames(GSE100426)[i],"stim")) condition[i,1]="stim"
  if(str_detect(colnames(GSE100426)[i],"nostim")) condition[i,1]="nostim"  
  if(str_detect(colnames(GSE100426)[i],"pl1")) plate[i,1]="plate1"
  if(str_detect(colnames(GSE100426)[i],"plate1")) plate[i,1]="plate1"
  if(str_detect(colnames(GSE100426)[i],"pl2")) plate[i,1]="plate2"
  if(str_detect(colnames(GSE100426)[i],"plate2")) plate[i,1]="plate2"
}
GSE100426$age=age
GSE100426$condition=condition
GSE100426$plate=plate
Idents(GSE100426) <- GSE100426$condition
GSE100426=subset(GSE100426,idents = "nostim")
Idents(GSE100426) <- GSE100426$plate

# Visualize QC metrics as a violin plot
VlnPlot(GSE100426, features = c("nFeature_RNA", "nCount_RNA"), ncol = 2)

GSE100426 <- subset(GSE100426, subset = nFeature_RNA > 1000 & nFeature_RNA < 6000)
GSE100426 <- NormalizeData(GSE100426, normalization.method = "LogNormalize", scale.factor = 10000)  
GSE100426 <- FindVariableFeatures(GSE100426, selection.method = "vst", nfeatures = 2000)  
GSE100426 <- ScaleData(GSE100426)
Idents(GSE100426) <- GSE100426$age

# find DEGs
markers <- FindAllMarkers(GSE100426,  logfc.threshold =0.6,test.use = "DESeq2", return.thresh=0.05)
save(markers,file="GSE100426_DESeq2_0.6_0.05_markers.Rdata")
write.csv(markers,"GSE100426.csv")
