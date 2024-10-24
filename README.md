DeepG  <a href="https://www.sri.inf.ethz.ch/"><img width="100" alt="portfolio_view" align="right" src="http://safeai.ethz.ch/img/sri-logo.svg"></a>
=============================================================================================================

DeepG is state-of-the-art system for certification of robustness of neural networks to geometric transformations. The key idea behind DeepG is to efficiently compute linear constraints of common geometric transformations. These constraints are then used as inputs to the state-of-the-art verifier [ERAN](https://github.com/eth-sri/eran/) which can then use these constraints to prove robustness of a neural network. 
The method is based on our [NeurIPS 2019](https://files.sri.inf.ethz.ch/website/papers/neurips19-deepg.pdf) paper and repository contains all code necessary to reproduce the results from the paper.
The system is developed at the [SRI Lab, Department of Computer Science, ETH Zurich](https://www.sri.inf.ethz.ch/) as part of the [Safe AI project](http://safeai.ethz.ch/).

>However, the initial tutorial is quite old, so I've provided an updated version. If you encounter any issues during running these commands, feel free to reach out to me on Slack or via email `m.ziyamova@yonsei.ac.kr`. Please note that I have also modified parts of the code to ensure everything works smoothly, even though you may still see many warnings :D
## Setup instructions

Clone this repository:

```bash
git clone https://github.com/iamafi/deepg.git
```

Download Gurobi, update environment variables and install C++ bindings:

>But before that you should obtain Gurobi Free Academic License. You can do this by using your university's network.

```bash
wget https://packages.gurobi.com/9.1/gurobi9.1.0_linux64.tar.gz
tar -xzvf gurobi9.1.0_linux64.tar.gz
rm gurobi9.1.0_linux64.tar.gz
export GUROBI_HOME="$(pwd)/gurobi910/linux64"
export PATH="${PATH}:${GUROBI_HOME}/bin"
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/lib:${GUROBI_HOME}/lib
grbgetkey <your key>
cd gurobi910/linux64/src/build
make
cp libgurobi_c++.a ../../lib/
sudo cp ../../lib/libgurobi91.so /usr/lib
cd ../../../../
```
>Please note that you may need to set up environment variables in every new session, unless you add them to the list of global environment variables.

Now you should be able to compile DeepG:

```bash
cd deepg/code
mkdir build
make deepg_constraints
```

Next, create a virtual environment, install Gurobi Python bindings and other required packages:

```bash
virtualenv venv
source venv/bin/activate
cd ..    # it's deepg folder
pip install -r requirements.txt
```
>Don't forget to activate virtual environment in new sessions.

Next, we install ERAN. Note that this is a fork of official [ERAN](https://github.com/eth-sri/eran/) analyzer.
As of now, ERAN will run only on CPU. GPU support will be added soon. The outputs will be the same, just the timing will be improved.
```bash
cd ERAN/
sudo ./install.sh
```


## Example of certification

As an example, we will certify that MNIST network is robust to rotation between -1 and 1 degrees for a single image.
First, we need to generate the constraints which capture the above rotation:

```bash
cd ../code
./build/deepg_constraints examples/example1
```

Then we need to download the network and run the verifier:

```bash
cd ../ERAN/tf_verify
wget https://files.sri.inf.ethz.ch/deepg/networks/mnist_1_rotation.pyt
python deepg.py --net mnist_1_rotation.pyt --dataset mnist --data_dir ../../code/examples/example1 --num_params 1 --num_tests 1
```

## Format of configuration file

Each experiment directory should have config.txt file which containts the following information:

```
dataset               dataset which we are working with, should be one of {mnist, fashion, cifar10}
noise                 L_infinity noise that should be composed with geometric transformations
chunks                number of splits along each dimension (each split is certified separately)
inside_splits         number of refinement splits for interval bound propagation
method                method to compute the constraints, to use DeepG it should be set to polyhedra
spatial_transform     description of geometric transformation in predefined format, see the examples
num_tests             number of images to certify
ub_estimate           estimate for the upper bound, usually set to Triangle
num_attacks           number of random attacks to perform for each image
poly_eps              epsilon tolerance used in Lipschitz optimization in DeepG
num_threads           number of threads (determines for how many pixels to compute the constraints in parallel)
max_coeff             maximum coefficient value in LP which DeepG is solving
lp_samples            number of samples in LP which estimates the optimal constraints
num_poly_check        number of samples in sanity check for soundness
set                   whether to certify test or training set
```

Parameter name and value are always separated by spaces. You can look at provided configuration files in constraints.zip to see the values used in our experiments.

## Reproducing the experiments

In order to reproduce the experiments from our paper, please download and unzip the constraints and configurations used in our experiments:

```bash
cd ../..  # it's deepg folder
wget https://files.sri.inf.ethz.ch/deepg/constraints.zip
unzip constraints.zip
rm constraints.zip
```

Each folder in folder `constraints` is named after one of the experiments in our paper. It contains constraints for 100 images.
To perform certification, you need to download the network with the same name as folder and then run deepg.py script.
Here is an example how to reproduce the results for the experiment with MNIST and translation:

```bash
cd ERAN/tf_verify
wget https://files.sri.inf.ethz.ch/deepg/networks/mnist_2_translation.pyt
python deepg.py --net mnist_2_translation.pyt --dataset mnist --data_dir ../../constraints/mnist_2_translation --num_params 2 --num_tests 100
```

You can also reproduce the constraints by running DeepG yourself (it will take some time). For example, the command for MNIST and translation is:

```bash
cd ../../code
./build/deepg_constraints ../constraints/mnist_2_translation
```

### Mini-note
>I hope you find this tutorial helpful. Once again, feel free to reach out to me if you have any questions. Happy coding!

Citing this work
---------------------

If you are using this library please use the following citation to reference this work:

```
@incollection{balunovic2019geometric,
	title = {Certifying Geometric Robustness of Neural Networks},
	author = {Balunovic, Mislav and Baader, Maximilian and Singh, Gagandeep and Gehr, Timon and Vechev, Martin},
	booktitle = {Advances in Neural Information Processing Systems 32},
	year = {2019}
}	
```

Contributors
------------

* [Mislav Balunović](https://www.sri.inf.ethz.ch/people/mislav)
* [Maximilian Baader](https://www.sri.inf.ethz.ch/people/max)
* [Gagandeep Singh](https://www.sri.inf.ethz.ch/people/gagandeep)
* [Timon Gehr](https://www.sri.inf.ethz.ch/people/timon)
* [Martin Vechev](https://www.sri.inf.ethz.ch/people/martin)

License and Copyright
---------------------

* Copyright (c) 2019 [Secure, Reliable, and Intelligent Systems Lab (SRI), ETH Zurich](https://www.sri.inf.ethz.ch/)
* Licensed under the [Apache License](http://www.apache.org/licenses/)





