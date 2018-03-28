# Building Julia On Cray and other Supercomputing Clusters
Sample scripts to build Julia on Cray and other supercomputing architectures

Since building Julia on Supercomputers is currently not well documented 
and I spent some time on it, I would like to share my experience on this page. 

This is a complimentary material for the workshop presented at [SuperComputing Frontiers 2018 conference](https://supercomputingfrontiers.eu/2018/tutorials-programme/).

I start by explaining how to build Julia on a computing cluster (based on Scientific Linux 6 / Cent OS 6.9) and then move forward to Cray (based on SLES 11). 

## Building Julia on a computing cluster

This part of the tutorial is based on my experience with [ICM](http://icm.edu.pl/en/)'s [Topola Supercomputer](https://kdm.icm.edu.pl/kdm/Topola)

Julia needs to be built separately for the access node (if needed) and for the workers nodes. 

Since the process takes few hours I advise to run `screen` before the following commands. 


### Building Julia for the access node

*Step 1* Find some directory with enough of free space (at least 4G per julia installation is required)


*Step 2* Clone and checkout
```
git clone https://github.com/JuliaLang/julia.git
cd julia
git checkout v0.6.2
```

*Step 3* Configure the `Make.user` file
```
echo "" >> Make.user
echo "USEICC = 0" >> Make.user
echo "USEIFC = 0" >> Make.user
echo "USE_INTEL_MKL = 1" >> Make.user
echo "USE_INTEL_LIBM = 0" >> Make.user
echo "" >> Make.user
```
*Step 4* configure the compilers
```
module use /icm/hydra/soft/easybuild/modules/all
module load GCC
module load M4
source /apps/compilers/intel/psxe2018u1/compilers_and_libraries_2018.1.163/linux/bin/compilervars.csh intel64
module load /apps/modulefiles/common/python/2.7.9
module load /apps/modulefiles/common/cmake/3.6.2
```

*Step 5* Run `make`
```
make
```


### Building Julia for the worker node node

Follow the steps 1-3 frome the previous tutorial

*Step 4* Run an interactive session on a worker node 

```
srun --pty --time 24:00:00 --mem=32GB -p topola bash -l
```

*Step 5* Configure the compilers

Note that since we are now running bash , we use different version of Intels's `compilervars` script
```
module use /icm/hydra/soft/easybuild/modules/all
module load GCC
module load M4
source /apps/compilers/intel/psxe2018u1/compilers_and_libraries_2018.1.163/linux/bin/compilervars.sh intel64
module load /apps/modulefiles/common/python/2.7.9
module load /apps/modulefiles/common/cmake/3.6.2
```

*Step 6* Run `make`
```
make
```




## Building Julia on a Cray XC40 Supercomputer 

This tutorial is based on my experience with [ICM](http://icm.edu.pl/en/)'s [Okeanos Cray XC40](https://kdm.icm.edu.pl/kdm/Okeanos)

*Step 1* Clone Julia and checkout
The main problem with building Julia on supercomputers is the outtdated software. Building Julia requires downloading several files from the the internet. In many scenarios you will see errors similiar to this one: `error:1407742E:SSL routines:SSL23_GET_SERVER_HELLO:`. The solution is to download Julia on some different machine and than compy it to the cluster access node with the `scp` command. For instructions how to clone see the previous tutorial. 



*Step 2* Download dependencies
Julia's builder downloads several files from the internet and will not be able to do so on Cray due to old SSL version. 
Firstly on the Cray access node go to your julia directory and create `deps/srccache/` subfolder, e.g.
```
cd julia
mkdir deps/srccache/
```
Now you need some some copy of Julia that you succesfully built on a different machine and run (from a remote machine):
```
scp julia_source_folder/deps/srccache/* user@hostname:/home/user/julia_target_folder/deps/srccache/
```
The above command will ignore folders - that is OK you need to only copy archive files. 

*Step 3* Configure the `Make.user` file
```
echo "" >> Make.user
echo "USEICC = 0" >> Make.user
echo "USEIFC = 0" >> Make.user
echo "USE_INTEL_MKL = 1" >> Make.user
echo "USE_INTEL_LIBM = 0" >> Make.user
echo "" >> Make.user



*Step 4* configure the compilers
```
module swap PrgEnv-cray PrgEnv-intel
module load PrgEnv-intel
module load gcc
module load  python/anaconda2.7
```

*Step 5* Install a newer cmake
```
bash contrib/download_cmake.sh




