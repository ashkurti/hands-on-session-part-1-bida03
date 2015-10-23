# Using the ExTASY workflows for enhanced sampling #

## Background ##
The example system that will be used for this hands on tutorial is a short peptide – BIDA03 (following figure) - related to the C-terminus of neuropeptide Y (NPY), and is of interest as a potential appetite suppressant for the treatment of obesity.

![01_bida03.jpg](https://bitbucket.org/repo/nGe76o/images/1966375726-01_bida03.jpg)

The C-terminus of NPY is believed to be critical to the binding to Y-receptors, but at present there is no structural data on this. As a result it is a popular target for simulation studies – see e.g. Ozer et al., JCTC 2010, 6, 3026-38.

## Analysis with conventional MD ##
Using a (relatively slow) conventional MD simulation approach (8 x 100ns simulations) we have analysed the conformational landscape of BIDA03.

Analysis of ensemble diversity as a function of simulation length:

![02_bida03_diversity.jpg](https://bitbucket.org/repo/nGe76o/images/2925245961-02_bida03_diversity.jpg)

PC0/PC1 sampling for full 8x100 ns ensemble:

![03_bida03_pcz_standard_md.jpg](https://bitbucket.org/repo/nGe76o/images/2923064886-03_bida03_pcz_standard_md.jpg)

## Enhanced sampling approach through the coco amber workflow of extasy ##
In this workshop you will use an enhanced sampling approach to study the same system. 

You can find the files that you will be using [here](https://bitbucket.org/extasy-project/hands-on-session-part-1-bida03/src/d63382b417eb11bff5ee93cf4da101563a8f3d49?at=bida03_1) and  can download and uncompress the tarball from your command line.

```
#!python

ssh username@workflow.iu.xsede.org

wget https://bitbucket.org/extasy-project/hands-on-session-part-1-bida03/get/bida03_5.tar.gz
tar -xvf bida03_5.tar.gz
cd extasy-project-hands-on-session-part-1-bida03-d6e36f301949

source /opt/tutorials/radical-tutorial/rp-tut.sh
virtualenv --system-site-packages $HOME/ve
source $HOME/ve/bin/activate
pip install radical.ensemblemd
```
You will now have learned how to run a standard ExTASY coco/amber workflow. For further instructions and help you can look [here]
(http://extasy.readthedocs.org/en/extasy_0.2/pages/coam.html)

```
#!python

RADICAL_ENMD_VERBOSE=REPORT python extasy_amber_coco.py --RPconfig stampede.rcfg --Kconfig cocoamber.wcfg
```

Note that we are now using a differing set of files that refer to the BIDA03 example. In this case you will run 4 iterations of 8 instances (replicas) each. On Stampede it would take approximately 5 minutes to run this job from the moment it starts running (not waiting anymore in the queue).

At the end of the workflow a folder named "output" will appear in your current directory:

```
#!python

ls -l output/*
output/iter1:
total 196
-rw------- 1 uname pa  1150 Oct 21 18:56 coco.log
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_1.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_2.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_3.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_4.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_5.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_6.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_7.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:55 md_1_8.ncdf

... 

output/iter4:
total 196
-rw------- 1 uname pa  1334 Oct 21 18:57 coco.log
-rw------- 1 uname pa 22488 Oct 21 18:56 md_4_1.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_2.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:56 md_4_3.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_4.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_5.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_6.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_7.ncdf
-rw------- 1 uname pa 22488 Oct 21 18:57 md_4_8.ncdf

```

To further analyze the input, the pyPcazip suite of analysis tools can be used. If you have not installed it locally, instructions on how to install it can be found [here](https://bitbucket.org/ramonbsc/pypcazip/wiki/Installation%20Guide). However, if you are running it from the workflow machine pyPcazip should already be installed there.

Run pyPcazip on the generated trajectories:

```
#!python

pyPcazip -t BIDA03dry.pdb -i output/iter?/*ncdf -o bida03_4_8.pcz
```

You can now explore the PC0/PC1 sampling for the generated trajectories after producing a Figure through the pyPczplot.py script. You will have in total 32 trajectory files each 0.01 ns long.

```
#!python

python pyPczplot.py bida03_4_8.pcz
```

You can now compare the sampling in the two cases also considering the length of the simulations in the two cases (traditional MD and coco-biased MD).

## Next steps ##

You can now modify the workload configuration file cocoamber.wcfg asking for a differing number of iterations as well as for a differing number of instances. The number of instances in this case is set to be equal to the number of computational units (num_CUs in the cocoamber.wcfg file). 
Pay attention: The PILOTSIZE parameter in the resource configuration file should be set as num_CUs * num_cores_per_sim_cu (both parameters of the workload configuration file).

You could try the same example asking for:
* 4 instances and 6 iterations
* 16 instances and 4 iterations
* etc.

And at the end compare the sampling of PC0/PC1 for the different cases.