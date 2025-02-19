## Accelerating Recommender Systems by 15x with RAPIDS (Source Code)
This repository contains demonstrations of the acceleration techniques used to accelerate the training of the fastai tabular deep learning model by a factor of 15x and the feature engineering of the inputs to the model by 9.7x during the RecSys 2019 Challenge where the team placed 15th/1534 teams.  Please follow the instructions carefully as additional files are currently a part of the repository that will be used for future versions.

## Prerequisites:
 - Collect the data at the following location: https://recsys.trivago.cloud/challenge/dataset/ (you need to sign up to get access)
 - [install the nightly version of RAPIDS(0.9a+)](https://rapids.ai/start.html)** (or use the nightly RAPIDS container), [PyTorch nightly](https://pytorch.org/get-started/locally/) and [fastai v1](https://docs.fast.ai/install.html)
 - A GPU capable of fitting the entire dataset in GPU memory (32GB Tesla V100).  We are working on versions that remove that restriction.

**Note that in it's current state feature engineering relies on cuDF 0.7 because of a [masking bug](https://github.com/rapidsai/cudf/issues/2141) while training requires 0.8.  This is resolved in the nightly build of cuDF and in 0.9+**

## Feature Creation
To give a point of comparison we've provided feature creation using both [rapids cuDF](https://github.com/rapidsai/dataloaders/tree/master/RecSys2019/FeatureEngineering/rapids) and [pandas](https://github.com/rapidsai/dataloaders/tree/master/RecSys2019/FeatureEngineering/pandas) which will hopefully help you translate your own feature engineering steps into cuDF.  

[create_data_pair_comparison_rapids.ipynb](https://github.com/rapidsai/dataloaders/blob/master/RecSys2019/FeatureEngineering/rapids/create_data_pair_comparison-rapids.ipynb) (or [it's pandas equivalent](https://github.com/rapidsai/dataloaders/blob/master/RecSys2019/FeatureEngineering/pandas/create_data_pair_comparison-panda.ipynb)) is the starting point for feature engineering and must be run before all other scripts.

Other examples of feature engineering are available in the [rapids](https://github.com/rapidsai/dataloaders/tree/master/RecSys2019/FeatureEngineering/rapids)  and [pandas](https://github.com/rapidsai/dataloaders/tree/master/RecSys2019/FeatureEngineering/pandas) folders but should not be run in the current iteration because they scale the dataset to a size that no longer fits in GPU memory when training.  A version of the preprocessing that handles larger than GPU memory datasets has been developed and is currently being refined.  It will be made available for RecSys 2019.

After you have successfully completed feature engineering. Verify the exported train.parquet, valid.parquet and test.parquet  files exist in the rsc19/cache/ folder. Once verified, you can proceed to preprocessing and model training. 

Note the version included does not require the LAMB optimizer; we were able to achieve similar performance with AdamW.  For deeper networks it likely will be a critical component.  The version of LAMB we used in our testing can be found [here](https://github.com/cybertronai/pytorch-lamb).

## Preprocessing and Training

The Training folder (currently in development) will contain the end to end workflows for an unoptimized version and [a GPU in memory version](https://github.com/rapidsai/dataloaders/blob/master/RecSys2019/Training/optimized_training_workflow_gpu.ipynb).  Setting 'to_cpu = True' on cell 7 of the GPU notebook modifies it so that the tensor is copied to CPU and dataloading happens from there.  Note that in order to see the full optimization effects you need to use the nightly version of PyTorch.  Without the kernel enhancement the model is ~6.5x slower.

## Future work

We are working on a larger than *CPU* memory version of the dataloader to handle cases of extreme datasets beyond the scope of the RecSys Competition.  When datasets scale beyond CPU memory the dataloading times are severely impacted and we're developing a solution that pre shuffles the data into randomized parquet files which are then further randomized upon load.  Stay tuned for more on this.

## Experiments

The experiments subfolder contains 3 notebooks, representing the three available model types. Choose one and run all cells to see speeds of each model across a range of batch sizes. 
