# Matilda: Multi-task learning from single-cell multimodal omics

Matilda is a multi-task framework for learning from single-cell multimodal omics data. Matilda leverages the information from the multi-modality of such data and trains a neural network model to simultaneously learn multiple tasks including data simulation, dimension reduction, visualization, classification, and feature selection.

<img width=100% src="https://github.com/liuchunlei0430/Matilda/blob/main/img/main.jpg"/>

Matilda is developed using PyTorch 1.9.1 and requires >=1 GPU to run.

## Installation
We recommend using conda enviroment to install and run Matilda. We assume conda is installed. The following installation code snippets were tested on a Ubuntu system.

Step 1:
Create and activate the conda environment for matilda
```
conda create -n environment_matilda python=3.8
conda activate environment_matilda
```

Step 2:
Check the environment including GPU settings and the highest CUDA version allowed by the GPU.
```
nvidia-smi
```

Step 3:
Install pytorch and cuda version based on your GPU settings.
```
# Example code for installing CUDA 11.3
conda install pytorch==1.9.1 torchvision==0.10.1 torchaudio==0.9.1 cudatoolkit=11.3 -c pytorch -c conda-forge
```

Step 4:
The following python packages are required for running Matilda: h5py, numpy, pandas, captum. They can be installed in the conda environment as below:
```
pip install h5py
pip install numpy
pip install pandas
pip install captum
```

Step 5:
Otain Matilda by clonning the github repository:
```
git clone https://github.com/liuchunlei0430/Matilda.git
```

## Preparing intput for Matilda
Matilda’s main function takes expression data (e.g., RNA, ADT, ATAC) in `.h5` format and cell type labels in `.csv` format. Matilda expects raw count data for RNA and ADT modalities. For ATAC modality, Matilda expects the 'gene activity score' generated by Seurat from raw count data.

An example for creating gene activity score from ATAC modality in the R environment using human gene annotation is as below:
```
gene.activities <- CreateGeneActivityMatrix2(peak.matrix=teaseq.peak,
                                             annotation.file = “Homo_sapiens.GRCh38.90.chr.gtf.gz”,
                                             seq.levels = c(1:22, “X”, “Y”),
                                             seq_replace = c(“:”))
```

### Example dataset
First,  we create the `./data` folder under `Matilda`:
```
cd Matilda
mkdir data
```

As an example, the processed TEA-seq dataset by Swanson et al. (GSE158013) is provided for the example run, which can be downloaded from [link](https://drive.google.com/file/d/1ojilvNBB95GbgtF9a0IHl3yWQwl6gDl8/view?usp=sharing) or the below command:
```
wget --load-cookies /tmp/cookies.txt "https://docs.google.com/uc?export=download&confirm=$(wget --quiet --save-cookies /tmp/cookies.txt --keep-session-cookies --no-check-certificate 'https://docs.google.com/uc?export=download&id=1ojilvNBB95GbgtF9a0IHl3yWQwl6gDl8' -O- | sed -rn 's/.*confirm=([0-9A-Za-z_]+).*/\1\n/p')&id=1ojilvNBB95GbgtF9a0IHl3yWQwl6gDl8" -O TEAseq.zip && rm -rf /tmp/cookies.txt
unzip TEAseq.zip
```

Users can prepare the example dataset as input for Matilda by downloading the dataset from the above link or use their own datasets.

## Running Matilda with the example dataset
### Training the Matilda model (see Arguments section for more details).
```
cd main
# training the matilda model
python main_matilda_train.py --rna [trainRNA] --adt [trainADT] --atac [trainATAC] --cty [traincty] #[training dataset]
```

### Argument
Training dataset information
+ `--rna`: path to training data RNA modality.
+ `--adt`: path to training data ADT modality (can be null if ATAC is provided).
+ `--atac`: path to training data ATAC modality (can be null if ADT is provided). Note ATAC data should be summarised to the gene level as "gene activity score".
+ `--cty`: path to the labels of training data.

Training and model config
+ `--batch_size`: Batch size (set as 64 by default)
+ `--epochs`: Number of epochs.
+ `--lr`: Learning rate.
+ `--z_dim`: Dimension of latent space.
+ `--hidden_rna`: Dimension of RNA branch.
+ `--hidden_adt`: Dimension of ADT branch.
+ `--hidden_atac`: Dimension of ATAC branch.

Other config
+ `--seed`: The random seed for training.
+ `--augmentation`: Whether to augment simulated data.

### Perform multiple tasks using trained Matilda model.
After training the model, we can use `main_matilda_task.py` to do multiple tasks with different augments.

### Argument for performing tasks
+ `--classification`: whether to do cell type classification.
+ `--fs`: whether to do cell type feature selection.
+ `--visualisation`: whether to do dimension reduction.
+ `--simulation`: whether to do simulation. 
+ `--simulation_ct`: whether to do cell type classification. Only be activated when `simulation = True`.
+ `--simulation_num`: whether to do cell type classification. Only be activated when `simulation = True`.


1) Multi-task on the training data

```
# using the trained model for data simulation
python main_matilda_task.py  --rna [trainRNA] --adt [trainADT] --atac [trainATAC] --cty [traincty] --simulation True --simulate_ct 1 -n 200
```
Output: The output will be saved in `./Matilda/output/simulation_result/TEAseq/`. To generate UMAP plots for the simulated data using R, run `./Matilda/qc/visualize_simulated_data.Rmd`. The UMAPs are:
<img width=50% src="https://github.com/liuchunlei0430/Matilda/blob/main/img/simulation_anchor.jpg"/> 

```
# using the trained model for data visualisation
python main_matilda_task.py  --rna [trainRNA] --adt [trainADT] --atac [trainATAC] --cty [traincty] --visualisation True
```
Output: The output will be saved in `./Matilda/output/visualisation/TEAseq/`. To generate UMAP plots and 4 clustering metrices, i.e., ARI, NMI, FM, Jaccard, for the latent space using R, run `./Matilda/qc/visualize_latent_space.Rmd`. The UMAPs are:
<img width=50% src="https://github.com/liuchunlei0430/Matilda/blob/main/img/visualisation.jpg"/> 

```
# using the trained model for feature selection
python main_matilda_task.py  --rna [trainRNA] --adt [trainADT] --atac [trainATAC] --cty [traincty] --fs
```
Output: The output will be saved in `./Matilda/output/fs/TEAseq/`. 


2) Multi-task on the query data
```
# using the trained model for classifying query data
python main_matilda_task.py  --rna [queryRNA] --adt [queryADT] --atac [queryATAC] --cty [querycty] --classification True
```

Output: The output will be saved in `./Matilda/output/classification/TEAseq/`.

```
The dataset is TEAseq
cell type :  0 	 	 prec : tensor(62.8241, device='cuda:0') number: 180
cell type :  1 	 	 prec : tensor(99.2557, device='cuda:0') number: 802
cell type :  2 	 	 prec : tensor(60., device='cuda:0') number: 11
cell type :  3 	 	 prec : tensor(98.6889, device='cuda:0') number: 639
cell type :  4 	 	 prec : tensor(70.6897, device='cuda:0') number: 37
cell type :  5 	 	 prec : tensor(97.1820, device='cuda:0') number: 283
cell type :  6 	 	 prec : tensor(45.4545, device='cuda:0') number: 12
cell type :  7 	 	 prec : tensor(68.1717, device='cuda:0') number: 1189
cell type :  8 	 	 prec : tensor(83.9875, device='cuda:0') number: 1020
cell type :  9 	 	 prec : tensor(84.3146, device='cuda:0') number: 576
cell type :  10 	 	 prec : tensor(83.3883, device='cuda:0') number: 299
The average classification accuracy is: tensor(77.6324)
```

```
# using the trained model for data simulation
python main_matilda_task.py  --rna [queryRNA] --adt [queryADT] --atac [queryATAC] --cty [querycty] --simulation True --simulate_ct 1 -n 200
```
Output: The output will be saved in `./Matilda/output/simulation_result/TEAseq/`. To generate UMAP plots for the simulated data using R, run `./Matilda/qc/visualize_simulated_data.Rmd`. 


```
# using the trained model for visualising query data
python main_matilda_task.py --rna [queryRNA] --adt [queryADT] --atac [queryATAC] --cty [querycty] --visualisation True
```

Output: The output will be saved in `./Matilda/output/visualisation/TEAseq/`. To generate UMAP plots and 4 clustering metrices, i.e., ARI, NMI, FM, Jaccard, for the latent space using R, run `./Matilda/qc/visualize_latent_space.Rmd`.  

```
# using the trained model for feature selection
python main_matilda_task.py --rna [queryRNA] --adt [queryADT] --atac [queryATAC] --cty [querycty] --fs True
```

Output: The output will be saved in `./Matilda/output/fs/TEAseq/`.


## Reference
[1] Ramaswamy, A. et al. Immune dysregulation and autoreactivity correlate with disease severity in
SARS-CoV-2-associated multisystem inflammatory syndrome in children. Immunity 54, 1083–
1095.e7 (2021).

[2] Ma, A., McDermaid, A., Xu, J., Chang, Y. & Ma, Q. Integrative Methods and Practical Challenges
for Single-Cell Multi-omics. Trends Biotechnol. 38, 1007–1022 (2020).

[3] Swanson, E. et al. Simultaneous trimodal single-cell measurement of transcripts, epitopes, and
chromatin accessibility using TEA-seq. Elife 10, (2021).
