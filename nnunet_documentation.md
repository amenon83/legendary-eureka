# nnUNet Documentation for Omen PC (Ubuntu)

## Table of Contents
- [Installation](#installation)
- [Running nnUNet](#running-nnunet)
  - [Upload Training and Prediction Data](#data-upload)
  - [Dataset Folder Setup](#dataset-setup)
  - [Preprocessing](#preprocessing)
  - [Training](#training)
  - [Prediction](#prediction)
  - [Plotting Predictions](#plotting-predictions)

## Installation

> **Note:** nnUNet is already installed. This section is for troubleshooting if issues arise. You can skip this section if you don't need it.

### Setup Process

1. **Prerequisites**
   - Make sure Spyder and Anaconda are installed

2. **Create Conda Environment**
   ```bash
   conda create -n nnunet_env python=3.10
   ```

3. **Activate Environment**
   ```bash
   conda activate nnunet_env
   ```

4. **Install PyTorch**
   ```bash
   pip3 install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cpu
   ```

5. **Install nnUNetv2**
   ```bash
   pip install nnunetv2
   ```

6. **Configure Environment Variables**
   - Edit `.bashrc` file:
     ```bash
     nano ~/.bashrc
     ```
   - Add these lines to the bottom:
     ```bash
     export nnUNet_raw="home/liyong/Desktop/nnUNet_Project_nnUNet_Data/nnUNet_raw"
     export nnUNet_preprocessed="home/liyong/Desktop/nnUNet_Project_nnUNet_Data/nnUNet_preprocessed"
     export nnUNet_results="home/liyong/Desktop/nnUNet_Project_nnUNet_Data/nnUNet_results"
     export nnUNet_compile=0
     ```

7. **Optional: Install HiddenLayer**
   ```bash
   pip install --upgrade git+https://github.com/FabianIsensee/hiddenlayer
   ```

8. **Configure Spyder**
   - Go to **Tools → Preferences → Python Interpreter**
   - Select "Use the following interpreter:"
   - Set path to: `/home/liyong/anaconda3/envs/nnunet_env/bin/python`

9. **Install Spyder Kernels**
   ```bash
   pip install spyder-kernels==3.0.0
   ```

## Running nnUNet

### Upload Training and Prediction Data

1. Open Files and navigate to `/home/liyong/Desktop/nnUNet_Project/nnUNet_Data`

2. **Upload Training Data**
   - Place training `.clog` file in the `Training_Data` folder
   - This data is what nnUNet will train on

3. **Upload Prediction Data**
   - Place prediction `.clog` file in the `Prediction_Data` folder
   - This test data is what nnUNet will make predictions on

### Dataset Folder Setup

1. **Create Dataset Folder**
   - Navigate to the `nnUNet_raw` folder
   - Create a new folder named `Dataset###_name`
     - `###` = 3-digit ID you assign to this training process
     - `name` = any descriptive name
     - Example: `Dataset001_tutorial`

2. **Open Spyder**
   - Launch through terminal (type 'spyder') or through the app

3. **Navigate to Scripts**
   - Click **Files** (middle right, among 'Help,' 'Variable Explorer,' 'Debugger,' 'Plots,' 'Files')
   - Navigate to `/home/liyong/Desktop/nnUNet_Project/Scripts`

4. **Configure Training Script**
   - Open `updated_nnu_code.py`
   - Update the following file path variables:
     - `merged_clog_files`: Full path to training `.clog` file
       ```python
       "/home/liyong/Desktop/nnUNet_Project/nnUNet_Data/Training_Data/{FILENAME}.clog"
       ```
     - `output_base`: Full path to your dataset folder
       ```python
       "/home/liyong/Desktop/nnUNet_Project/nnUNet_Data/nnUNet_raw/Dataset###_name"
       ```

5. **Run Script**
   - Press the green run button at the top of the Spyder IDE
   - You can check that the folder named `Dataset###_name` has the following:
     - `imagesTr` folder
     - `labelsTr` folder
     - `dataset.json` file

### Preprocessing

1. **Open Terminal**
   ```bash
   ctrl + alt + T
   ```

2. **Activate Environment**
   ```bash
   conda activate nnunet_env
   ```

3. **Run Preprocessing**
   ```bash
   nnUNetv2_plan_and_preprocess -d ### --verify_dataset_integrity
   ```
   - Replace `###` with your 3-digit dataset ID

4. **Verify Success**
   - Check for new `Dataset###_name` folders in:
     - `nnUNet_Data/nnUNet_preprocessed`
     - `nnUNet_Data/nnUNet_results`

### Training

1. **Optional: Monitor Memory Usage**
   - You can open a separate terminal and use htop to monitor the PC's memory while training
   ```bash
   htop
   ```

3. **Start Training**
   ```bash
   nnUNetv2_train ### 2d 0
   ```
   - Replace `###` with your dataset ID

4. **Monitor Progress**
   - Training continues until EMA pseudo Dice score stops improving
   - It'll say "Yayy! New best EMA pseudo Dice: ..." each time the Dice score improves
   - nnUNet runs up to 1000 epochs by default
   - Training on the Omen PC is slow - stop when the Dice score no longer improves consistently

5. **Stop Training**
   - Press `Ctrl + C` (you might have to do this twice to stop fully)

6. **Rename Checkpoint File**
   - Navigate to: `/home/liyong/Desktop/nnUNet_Project/nnUNet_Data/nnUNet_results/Dataset###_name/nnUNetTrainer__nnUNetPlans__2d/fold_0`
   - Rename `checkpoint_best.pth` to `checkpoint_final.pth`
   - **Why:** This file is only automatically renamed when Epoch 1000 is reached, but we usually don't train that long

### Prediction

1. **Verify Prediction Data**
   - Ensure correct prediction `.clog` file is in `/home/liyong/Desktop/nnUNet_Project/nnUNet_Data/Prediction_Data`

2. **Create Output Folder**
   - Navigate to `/home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/`
   - Create a new folder with some descriptive name (e.g. `tutorial`)
   - This new folder will store prediction NIfTI images (`tutorial/images` and `tutorial/predictions`)

3. **Configure Prediction Script**
   - In Spyder, navigate to `/home/liyong/Desktop/nnUNet_Project/Scripts`
   - Open `Predict_to_NIFTI.py`
   - Update variables:
     - `clog_filepath`: Full path to prediction `.clog` file
       ```python
       "/home/liyong/Desktop/nnUNet_Project/nnUNet_Data/Prediction_Data/{FILENAME}.clog"
       ```
     - `output_dir`: Path to new folder with `/images` subfolder
       ```python
       "/home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/{FOLDER_NAME}/images"
       ```
       > **Important:** Include `/images` after folder name. The script will automatically create this subfolder

4. **Run Script**
   - Execute the script
   - You can check that the `images` folder is populated

5. **Generate Predictions**
   ```bash
   nnUNetv2_predict -d ### -i <path_to_prediction_images> -o <path_to_output> -c 2d -f 0
   ```
   
   **Example:**
   ```bash
   nnUNetv2_predict -d ### -i /home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/{FOLDER_NAME}/images -o /home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/{FOLDER_NAME}/predictions -c 2d -f 0
   ```
   
   - The `/predictions` folder will be created automatically

### Plotting Predictions

1. **Configure Visualization Script**
   - In Spyder, navigate to `/home/liyong/Desktop/nnUNet_Project/Scripts`
   - Open `nnunet_prediction_visualizer.py`
   - Update variables:
     - `image_dir`: Path to images folder
       ```python
       "/home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/{FOLDER_NAME}/images"
       ```
     - `pred_dir`: Path to predictions folder
       ```python
       "/home/liyong/Desktop/nnUNet_Project/Prediction_NIfTI_Images/{FOLDER_NAME}/predictions"
       ```

2. **Run Visualization**
   - Execute the script
   - Console should display: "Visualizing: Frame..."

3. **View Results**
   - Click **Plots** (middle right, among 'Help,' 'Variable Explorer,' 'Debugger,' 'Plots,' 'Files')
   - View prediction plots as script runs
