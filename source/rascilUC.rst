================================
Use-case workflows and datasets
================================


Use-case workflows
*******************

.. _MeerKAT_Point_Source:

MeerKAT Point Source Catalog
=================

This guide is both an example of how to create containers which can be used to process survey data and a guide on using IRIS resources to process the MeerKAT point source catalog and spectral index fitting.

.. _MeerKAT-Point-Source-getting-started:

Building the Singularity Container
---------------

The basics of creating a Singularity container are covered in :ref:'use-of-sunglarity', however the example given focuses on create a CASA container. We want to create a container that uses a specific python version, python packages, and custom python programs and collects them in a container so anyone can reproduce the same results when processing MeerKAT data.

Load the singularity module by entering:

	.. code-block:: console

		(host) $ module load singularity

The MeerKAT singularity container takes a command line input to run one of several data-reduction python programs.

Creating Python Scripts to Run Slurm
-----------

The MeerKAT processing python programs are designed to run on induvidual data cubes. This is due to several of the programs taking 1-4 hours to processes a cube due to the large number of point sources.

The code below is an example of how to write a python script which will submit a number of jobs to slurm. You will need both a python program submitting the jobs as well as a slurm file providing the proper. Note, this file is NOT within the container. It is an outside program so that we can run the container multiple times.
	
	.. code-block:: console

		File Edit Options Buffers Tools Python Help                                                                                                                                                                  
		#!/usr/bin/env python3                                                                                                                                                                                       
		# -*- coding: utf-8 -*-                                                                                                                                                                                      

		import os
		import subprocess

		# =============================================================================                                                                                                                              
		# Top layer python script to set multiple jobs going on the cluster.                                                                                                                                         
		# =============================================================================                                                                                                                              

		filepath = '/rds/project/rds-bRdYdViqoGA/bsmart/MeerKAT/Mosaic_Planes/'
                                                                                                                                                         
		f=os.listdir (filepath)

		for filename in f:

        	folder=filepath+filename+'/'
        	sbatch_command = """sbatch --export=folder='{0}' /rds/project/rds-bRdYdViqoGA/bsmart/MeerKAT/Singularity/run_bane.sh""".format(folder)
        	print(sbatch_command)
        	print(folder)
        	print('Submitting job')
        	exit_status = subprocess.call(sbatch_command, shell=True)

        	if exit_status is 1:
                	print('Job failed to submit')

			print('Done submitting jobs!')
	
The above program imports both os and suprocess so that we can use console commands to submit multiple slurm. It takes a filepath to a folder containing all the meerkat cubes and sends each cube as an input to a Singularity container.
	
Creating Slurm Job Submission file
-----------	
	
The bellow code will be an example of a slurm job submission file which will be 
	
	.. code-block:: console

		#!/bin/bash                                                                                                                                                                                              
		#SBATCH -A DIRAC-TP001-CPU                                                                                                                                                                               
		#SBATCH -p skylake                                                                                                                                                                                       
		#SBATCH --ntasks 32                                                                                                                                                                                      
		#SBATCH --time=36:00:00                                                                                                                                                                                  
		#SBATCH --output=banetest_%j.log                                                                                                                                                                         
		#SBATCH --mail-type=ALL                                                                                                                                                                                  
		#I) tasks will there be in total? (<= nodes*32)                                                                                                                                                          

		#! The skylake/skylake-himem nodes have 32 CPUs (cores) each.                                                                                                                                            

		#! Number of nodes and tasks per node allocated by SLURM (do not change):                                                                                                                                

		numnodes=$SLURM_JOB_NUM_NODES
		numtasks=$SLURM_NTASKS
		mpi_tasks_per_node=$(echo "$SLURM_TASKS_PER_NODE" | sed -e  's/^\([0-9][0-9]*\).*$/\1/')

		#! Optionally modify the environment seen by the application                                                                                                                                             

		#! (note that SLURM reproduces the environment at submission irrespective of ~/.bashrc):                                                                                                                \           
		. /etc/profile.d/modules.sh                # Leave this line (enables the module command)  
		module purge                               # Removes all modules still loaded 
		module load rhel7/default-peta4            # REQUIRED- loads the basic environment 
		module load singularity
		pwd; hostname; date
		FILENAME=${folder}
		#! Full path to application executable:                                                                                                                                                                  
		application="singularity run -B/rds/project/rds-bRdYdViqoGA/bsmart/MeerKAT meerkat_test.sif"
		
		#! Run options for the application:                                                                                                                                                                      
		options="python3 /usr/local/MeerKAT/python_programs/auto_bane_cluster.py --input_folder=${FILENAME}"

		#! Work directory (i.e. where the job will run):                                                                                                                                                         
		workdir="/rds/project/rds-bRdYdViqoGA/bsmart/MeerKAT/Singularity/"  # The value of SLURM_SUBMIT_DIR sets workdir to the directory                                                                        

                             # in which sbatch is run.                                                                                                                                                   
		#! Are you using OpenMP (NB this is unrelated to OpenMPI)? If so increase this             

This particular job requires a path to the data be provided. The previous folder variable that was in the python program is avalaible to the slurm script.

To run singularity using slurm, we need to load the singularity module within the slurm script. In the above script, this is done with the following lines.

	.. code-block:: console
		. /etc/profile.d/modules.sh                # Leave this line (enables the module command)  
		module purge                               # Removes all modules still loaded 
		module load rhel7/default-peta4            # REQUIRED- loads the basic environment 
		module load singularity
		
After singularity is loaded in the slurm script, any number of processes in the singularity container can be run. In this example, we are giving slurm the singularity command as an application. The singularity command loads in the desired container, and options then passes the commands to the container so that we can run the desired applications within the container. In this example, we are asking the container to run the program auto_bane_cluster.py in python3 with an input variable "input_folder". In this example, ${FILENAME} is a variable which was passed from the original python3 script outside the container and into this script so multiple jobs can be run on different files. 

	.. code-block:: console
		#! Full path to application executable:  
		application="singularity run -B/rds/project/rds-bRdYdViqoGA/bsmart/MeerKAT meerkat_test.sif
		
		#! Run options for the application:  
		options="python3 /usr/local/MeerKAT/python_programs/auto_bane_cluster.py --input_folder=${FILENAME}"


Running MeerKAT Data Processing
-----------
Processing the MeerKAT data from the cubes is split up into several different programs and is dependant on three file locations. For this example, I have a master folder called MeerKAT which contains all of the data needed to create the spectral index catalog.
 

File Setup
-----------
+ To processes the MeerKAT data, you need a folder which contains the following files:
	- Aegean_Test_Catalogue_Full
	- Mom0_comp_catalogs
	- Mosaic_Planes
	- Singularity
	- python_scripts

The first three folders are required as they contain all of the relevant MeerKAT data that has been processed. The Aegean_Test_Catalogue_Full contains the folders:
	- Mom0_comp_catalogs  
	- Mom0_comp_ds9_regions  
	- Mom0_isle_catalogs  
	- Mom0_isle_ds9_regions

and can be found by accessing PATH_HERE

The Mom0_comp_catalogs folder contains all of the moment zero maps of the cubes. These are used to process the average background used.
	- Mom0_comp_catalogs  
	- Mom0_comp_ds9_regions  
	- Mom0_isle_catalogs  
	- Mom0_isle_ds9_regions

Process Background
-----------
 
 The first step to processing the MeerKAT data cubes is to create the backgrounds for the 0th moment maps. The induvidual backgrounds for each plane have been seperately processed, and the combined zeroth moment is needed to background subtract. The background is processed with the BANE program, part of the Aegean processing package used to make the full point source catalog created by Mubela Mutale and found on the MeerKAT survey repository.
 
 	.. code-block:: console
		python3 jobSubmitter_Bane.py

This job submitter is the same one used in the example above, and sends each induvidual cube file path to a singularity container submitted to slurm to process the background.

Process Photometric Catalog
-----------
Once the backgrounds have been processed, run the following program.

 	.. code-block:: console
		python3 jobSubmitter_Phot.py

This program submits induvidual cubes to slurm, where it reads in the Aegean point source catalog and uses the Bane backgrounds and catalog to measure the photometry for each wavelength in the cube using astropy photometry. These are written to indivudial photometry files for each layer.

Process Spectral Indices and Clean Up Catalog
-----------

Now that the photometry at each wavelength has been calculated, we can put together a spectral index catalog for each of the induvidual points. The following programs throw out points which are not bright enough and ones which do not have enough measurements in each wavelength.
 

Merging Catalogs
-----------
After all the data has been processed and the catalog columns have been organized and cleaned, this program takes all of the different induvidual cube catalogs and merges them into one large catalog, giving each source a designated reference number in the process.

Best Practice Notes
-----------

There are several things you want to keep in mind when creating a container.

- You want to keep your container as small as possible. The idea is you are creating a purpose built containerised environment that can process your data the way you want, and nothing else. This also means your data should not be within the container. It will be passed in outside of the container, and the results will be processed outside the container.
- Make sure all of the programs inside your container have generalized paths. It is better to pass in a path to your data rather than have it coded in. This allows more flexibility.



LOFAR processing with ddf-pipeline
==================================

ddf-pipeline <https://github.com/mhardcastle/ddf-pipeline> is the
LOFAR Surveys Key Science Project standard reduction pipeline for
Dutch-baseline data. It is designed for reduction of data which has
already been processed through prefactor
<https://git.astron.nl/eosc/prefactor3-cwl>. It also contains a number
of utility routines for working with the archived LoTSS data.

Setup
-----

ddf-pipeline comes with a singularity recipe. You can build the
singularity image from the recipe
<https://github.com/mhardcastle/ddf-pipeline/blob/master/ddf-py3.singularity>
or pull from the singularity repository:

       .. code-block:: console
		       
		       (host) $ singularity pull library://mhardcastle/default/ddf-pipeline

Interactive use
---------------

Use ``singularity shell`` to have access to a shell in which you can
run commands in the ddf-pipeline environment.

      .. code-block:: console

		      (host) $ singularity shell ddf-pipeline_latest.sif

For example here you can run data preparation commands such as
``download_field.py`` and ``make_mslist.py``.
		      
Use in jobs
-----------

The following is a basic Slurm script to run ddf-pipeline itself from
the singularity environment. You'll need to adjust the working
directory and other environment variables.

.. code-block:: console

		#!/bin/bash

		#! Call with --export FIELD=Pxxx+xx

		#SBATCH -J ddf-pipeline
		#SBATCH --nodes=1
		#SBATCH --ntasks=32
		#SBATCH --time=36:00:00
		#SBATCH --mail-type=ALL
		#SBATCH --no-requeue
		#SBATCH -p skylake-himem

		. /etc/profile.d/modules.sh
		module purge               
		module load rhel7/default-peta4 
		module load singularity
		
		application="singularity run ddf-pipeline-latest.sif"
		options="pipeline.py tier1-config.cfg"
		workdir="/rds/project/rds-bRdYdViqoGA/mjh/$FIELD"
		
		export OMP_NUM_THREADS=1
		
		export DDF_PIPELINE_CATALOGS=/rds/project/rds-bRdYdViqoGA/mjh/bootstrap
		export DDF_PIPELINE_DATABASE=True
		export DDF_PIPELINE_CLUSTER=Cambridge

		CMD="$application $options"

		cd $workdir

		JOBID=$SLURM_JOB_ID

		echo -e "JobID: $JOBID\n======"
		echo "Time: `date`"
		echo "Running on master node: `hostname`"
		echo "Current directory: `pwd`"

		echo -e "\nExecuting command:\n==================\n$CMD\n"

		eval $CMD 

The script illustrates the use of an environment variable to determine
what field will be processed. Call with


      .. code-block:: console

		      (host) $ sbatch --export FIELD=P123+45 ddf-pipeline.sh

Note that ideally ddf-pipeline requires a week's walltime, so you
should be prepared for several restarts if the local system does not
allow this. You can use the ``--dependency=afterany:JOBID`` option to
``sbatch`` to stack up a set of dependent jobs that will give the
required walltime.
		      

Self-calibration
----------------

This section describes how to use the self-calibration scripts to
improve the calibration of public or private LoTSS data. [TBD]




Datasets
*********

ALMA Datasets
==============

ALMA Datasets are downloaded from `almascience <https://almascience.nrao.edu/alma-data/science-verification>`_ and uploaded under the file catalog (FC) IRIS under FC:/skatelescope.eu/user/c/cimpan/rascil

#. CY4234_L_004_A262-1_20170123_avg_0316+4119.tar.gz
#. HLTau_Band3_CalibratedData.tgz
#. HLTau_Band3_cn_CalibratedData.tgz
#. HLTau_Band3_co_CalibratedData.tgz
#. HLTau_Band4_X1389_CalibratedData.tgz
#. HLTau_Band4_X15e2_CalibratedData.tgz
#. HLTau_Band4_X7b8_CalibratedData.tgz
#. HLTau_Band4_Xa09_CalibratedData.tgz
#. HLTau_Band6_CalibratedData.tgz
#. HLTau_Band7_CalibratedData.tgz
#. HLTau_Band9_X1b89_CalibratedData.tgz
#. HLTau_Band9_X2046_CalibratedData.tgz
#. Mira_Band3_CalibratedData.tgz
#. Mira_Band6_CalibratedData.tgz
#. NGC3256_Band3_CalibratedData.tgz
#. TWHYA_BAND7_CalibratedData.tgz

Any dataset can be accessed via **InputData** in a .jdl job submission. See `IRIS through certificate <https://irisdocumentation.readthedocs.io/en/latest/JobSub.html#iris-through-certificate>`_ for more details.


