```conda create -n xxx python=3.x ```： 安装新的conda环境

```conda create -n xxx --clone AAA```：复制conda环境

```conda activate xxx```：启动环境

```conda env list```

```conda remove -n xxx --all```

```conda deactivate```：退出当前环境







```shell
#!/bin/bash
SBATCH --job-name=download_dataset             # Name of your job
#SBATCH --output=%x_%j.out            # Output file (%x for job name, %j for job ID)
#SBATCH --error=%x_%j.err             # Error file
SBATCH --partition=CPU              # Partition to submit to (A100, V100, etc.)
#SBATCH --cpus-per-task=8             # Request 8 CPU cores
#SBATCH --mem=32G                     # Request 32 GB of memory
SBATCH --time=48:00:00               # Time limit for the job (hh:mm:ss)

# Print job details
echo "Starting job on node: $(hostname)"
echo "Job started at: $(date)"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garment_posed_data_v2.json"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garment_posed_data_v3.json"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garment_posed_data_v4.json"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garment_restpose_data_v1.json"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v2_1.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v2_2.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v2_3.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v3.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v4.zip"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/gpt4o_prompts.txt"

mkdir new_garments
cd new_garments
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/new_garments/garment_v3_1.zip"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/new_garments/garment_v3_2.zip"

wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/new_garments/garment_v4.zip"

cd ..
mkdir evaluations
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/evaluations/close_eva_imgs.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/evaluations/dress4d_eva_imgs.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/evaluations/garment_edit_eva.json"

cd ..
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v1_1.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v1_2.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v1_3.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v1_4.zip"
wget --content-disposition "https://huggingface.co/datasets/sy000/ChatGarmentDataset/blob/main/garments_imgs_v1_5.zip"
```

