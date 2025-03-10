# FACMIC
Official codebase for FACMIC: Federated Adaptative CLIP Model for Medical Image Classification (Accepted at MICCAI 2024)

Authors: Yihang Wu, Christian Desrosiers and Ahmad Chaddad

We provide codes based on BrainTumor experiments using FACMIC (you can change to other methods following the steps listed below), you can simulate directly using main.py.

## Requirements

We suggest you to use the following packages:

clip==1.0

numpy==1.22.0

opencv-python==4.9.0.80

openpyxl==3.1.2

Pillow==9.3.0

scikit-image==0.21.0

scikit-learn==1.1.3

scipy==1.10.0

tqdm==4.66.1

torch==1.13.1+cu117

torchvision=0.14.1+cu117

```
conda create -n facmic_env python=3.8
conda activate facmic_env

pip install git+https://github.com/openai/CLIP.git

pip install numpy==1.22.0
pip install opencv-python==4.9.0.80
pip install openpyxl==3.1.2
pip install Pillow==9.3.0
pip install scikit-image==0.21.0
pip install scikit-learn==1.1.3
pip install scipy==1.10.0
pip install tqdm==4.66.1

conda install pytorch==1.13.1 torchvision==0.14.1 pytorch-cuda=11.7 -c pytorch -c nvidia
```

## How to use

### main.py 

Run main.py to reproduce our results.

parser.add_argument('--test_envs', type=int, nargs='+', default=[3]) # default here is to set the global testing set, suppose there are 4 Clients, 3 here means it will treat Client 4 as the global while the rest as training clients.

### adaptation.py

adaptation.py is the domain adaptation technique. You have to change the classes in Line 64: self.num_class = 7 (For Real is 7, for BT is 4).

### nets/models.py

models.py is the model backbone file. In Line 46, you have to change attention module if you want to test FedCLIP( just comment Line 47-48, use Line 51-52)

### utils/clip_util.py

clip_util.py is the utils that CLIP will use. For FedAVG, MOON and FedProx, you have to set Line 23 True, For FACMIC and FedCLIP, you have to set it as False.

### utils/prepare_data_dg_clip.py

prepare_data_dg_clip.py is the dataloader CLIP will use. You can define the percentage for training, val and test in Line 97.


### utils/training.py

training.py is the training function for all methods. For SC and BT dataset, you have to use the following statement:

```sh
if i == args.n_iter:
    break
```

This can ensure the funciton will stop in right way. We marked these sentences in Line 32-33, Line 82-83, 124-125, Line 160-161, Line 211-212 and Line 241-242. Besides, you have to change train(args, model, train_test_loaders[client_idx], optimizers[client_idx], device, test_train[0], mmd_loss, server_model_pre, previous_nets[client_idx]) into train(args, model, train_loaders[client_idx], optimizers[client_idx], device, test_train[0], mmd_loss, server_model_pre, previous_nets[client_idx]) in main.py. Only for Real dataset, you have to use train_test_loaders[client_idx], and comment these statements mentioned above (e.g., Line 32-33).


## dataset

Here is a case study about the structure of our dataset as follows:

```sh
./data/BrainTumor/
    client_0/
      glioma_tumor/
        gg (1).jpg
            ...
      meningioma_tumor/
      no_tumor/
      pituitary_tumor/
    client_1/
    client_2/
    client_3/
```
We provide BrainTumor data as an example. you can download it via https://drive.google.com/drive/folders/1__sN_2857bnhjJdEoBR48dweXRqM_ZsA?usp=sharing.

## Run

``cd "Project-Code/Fed-Learning/FED-FACMIC-code" ``

Run with default parameters

``python main.py``

Run with custom parameters

``python main.py --batch 16 # To fasciliate less memory``

``python main.py --dataset BrainTumor --n_clients 4 --test_envs 3 --iters 50 --wk_iters 1``

## N. B.

Important Code Flow:

1. The code initializes a CLIP model for each client
2. Uses client 3 (index [3]) as the global test set by default
3. Performs federated learning across clients 0, 1, and 2
4. Uses CLIP embeddings for feature extraction
5. Implements domain adaptation using LMMD loss
6. Aggregates models using attention-based mechanism


Key Requirements:

1. CLIP
2. PyTorch
3. The dataset should be organized in client folders
4. Each client folder should have the same class structure

Remember to update ``adaptation.py`` to set the correct number of classes (4 for BrainTumor dataset) and adjust ``clip_util.py`` settings based on which federated learning method you're using (FACMIC, FedCLIP, etc.).
