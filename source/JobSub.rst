=====================================================================
Job submission on IRIS Resources - Galahad, Dirac SAFE and IRIS(cert)
=====================================================================



Galahad
========

Job submission:
---------------

To submit a job on Galahad:

.. code:: python

   [<your-user>@galahad ~]$ cat  slrascil1.sh
   #!/bin/bash
   #SBATCH --ntasks 1
   #SBATCH --time 5:0
   #SBATCH --output=test_%j.log
   pwd; hostname; date

   module load python37base gcc920
   CMD="singularity exec /home/<your-user>/RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py"
   eval $CMD


 - Submit the job using the command:
   
   [<your-user>@galahad ~]$  sbatch slrascil1.sh
   Submitted batch job 3404


 - Check the submitted job:

   [<your-user>@galahad ~]$  squeue
   JOBID PARTITION     NAME     USER ST       TIME  NODES NODELIST(REASON)
   3404   CLUSTER slrascil   <your-user>R       0:18      1 compute-0-7
   
   
Note: RASCIL-full1.img can be found under /share/nas/cimpan



VNC on Galahad
---------------


.. code:: python

   You run vncserver on galahad (already installed). On your windows PC use:
   https://www.tightvnc.com/download-old.php as your vnc viewer.

   When you run vncserver for the first time you will set up a password. 
   It will report it has created a virtual display galahad.ast.man.ac.uk:X
   The X will be a number. You then use that address in your vnc viewer

   [<your-user>@galahad ~]$ vncserver
   [<your-user>@galahad ~]$ vncserver -kill :3
   Killing Xvnc process ID 35841

With vnc I would suggest editing the default .vnc/xstartup file (created
after you run vncserver for the first time) to change the last line to
run /usr/bin/icewm as the window manager rather than xinitrc. You should
then kill off your first vncserver and run it again to pick up the
change. This avoids a bug where sometimes the VNC just displays a black
screen.

.. code:: python


   [<your-user>@galahad ~]$ cat .vnc/xstartup
   #!/bin/shunset SESSION_MANAGER
   unset DBUS_SESSION_BUS_ADDRESS
   #exec /etc/X11/xinit/xinitrc
   /usr/bin/icewm
   [<your-user>@galahad ~]$ vncserver #restarting the server

How to find the host for the for the diagnostics page? It would be
whichever host has started it, so use squeue to see what host is running
your job and then it would be for example http://compute-0-5:8787

.. code:: python

   [<your-user>@galahad ~]$ squeue



   
   
Dirac SAFE 
===========


To submit a job on  Dirac SAFE (skylake):

.. code:: python

   [<your-user>@login-e-13 ~]$ cat  slrascil1.sh
   #!/bin/bash
   #SBATCH -A DIRAC-TP001-CPU
   #SBATCH -p skylake
   #SBATCH --ntasks 1
   #SBATCH --time 5:0
   #SBATCH --output=test_%j.log
   pwd; hostname; date

   CMD="singularity exec /home/<your-user>/RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py"
   eval $CMD


 - Submit the job using the command:
   
   [<your-user>@login-e-13 ~]$ sbatch slrascil1.sh
   Submitted batch job 52726369
   
   
 - Check the submitted job:
   
   [<your-user>@login-e-13 ~]$ squeue | grep <your-user>
   52726369   skylake slrascil <your-user>  R       0:04      1 cpu-e-820

   
 - Check the results:
   
   [<your-user>@login-e-13 ~]$ ls
   imaging_dirty.fits  imaging_restored.fits  
   imaging_psf.fits   
   
   
 - Check the logfile:
   
   [<your-user>@login-e-13 ~]$ cat test_52726369.log
   
   
   
   

IRIS through certificate 
=========================

To get access to IRIS and submit jobs, please follow the documentation under `Signing up to IRIS through certificate <https://irisdocumentation.readthedocs.io/en/latest/signupIR.html#signing-up-to-iris-through-certificate>`__  that gives details how to get a certificate, be approved by a VO and install DIRAC in order to be able to submit jobs to IRIS - jdl and py forms.

From the server where dirac is installed:

-  start proxy before using any dms commands

   .. code:: python

          bash-4.2$ /raid/scratch/<your-user>/dirac_ui > source bashrc
          bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-proxy-init -x -N

-  Add the RASCIL container to the filecathalog using command
   "dirac-dms-add-file"

   .. code:: python

      dirac-dms-add-file LFN:/skatelescope.eu/user/c/<your-user>/rascil/RASCIL-full1.img RASCIL-full1.img  UKI-NORTHGRID-MAN-HEP-disk

-  check where the file has been uploaded using command
   "dirac-dms-filecatalog-cli"



Submit a simple job
--------------------

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > cat simple.jdl
   JobName = "InputAndOuputSandbox";
   Executable = "pythonV.sh";
   StdOutput = "StdOut";
   StdError = "StdErr";
   InputSandbox = {"pythonV.sh"};
   OutputSandbox = {"StdOut","StdErr"};

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > cat pythonV.sh
   #!/bin/bash
   /usr/bin/python --version;


Monitor a simple job
---------------------

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-wms-job-submit simple.jdl
   JobID = 25104301

   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-wms-job-status 25104301
   JobID=25104301 Status=Done; MinorStatus=Execution Complete;
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

- The job execution can be seen also on DIRAC `Web-link <https://dirac.gridpp.ac.uk:8443/DIRAC/>`__
(see Applications/Job Monitor -> Owner (your name) -> submit)

Put RASCIL.img in a file catalog
---------------------------------

**Details at:**  `File_Catalog <https://dirac.readthedocs.io/en/latest/UserGuide/CommandReference/DataManagement/index.html>`__

.. code:: python
   
   - Accessing File Catalog to add a testFile or a singularity container
   bash-4.2$ dirac-dms-filecatalog-cli
   Starting FileCatalog client
   FC:/> cd /skatelescope.eu/user
   
   - Go to the first letter of your user 
   FC:/skatelescope.eu/user>cd c
   
   - Create (mkdir) or go to your user folder
   FC:/skatelescope.eu/user/c>cd cimpan
   
   - Exit File Catalog
   FC:/skatelescope.eu/user/c/cimpan>exit
   bash-4.2$ 
   
   bash-4.2$ /raid/scratch/<your-user>/dirac_ui > dirac-dms-add-file LFN:/skatelescope.eu/user/<first letter of your user>/<your-user>/rascil/RASCIL.img RASCIL.img UKI-NORTHGRID-MAN-HEP-disk
   # UKI-NORTHGRID-MAN-HEP-disk - SE: DIRAC Storage Element

   Then you will find the file RASCIL.img under: 
   FC:/skatelescope.eu/user/<first letter of your user>/<your-user>/rascil/RASCIL.img

Submitting RASCIL job
----------------------

.. code:: python

   cat simpleR1.jdl
   JobName    = "InputAndOuputSandbox";
   Executable = "testR1.sh";
   StdOutput = "StdOut";
   StdError = "StdErr";
   InputSandbox = {"testR1.sh"};
   InputData = {"LFN:/skatelescope.eu/user/c/cimpan/rascil/RASCIL-full1.img"};
   OutputSandbox = {"StdOut","StdErr","imaging_dirty.fits","imaging_psf.fits","imaging_restored.fits"};
   OutputSE ="UKI-NORTHGRID-MAN-HEP-disk";
   Site = "LCG.UKI-NORTHGRID-MAN-HEP.uk";

    cat testR1.sh
   #!/bin/bash
   singularity exec --cleanenv -H $PWD:/srv --pwd /srv -C RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py;

Managing RASCIL job
--------------------

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   $ dirac-wms-job-submit simpleR1.jdl
   JobID = 25260750

   $ dirac-wms-job-status  25260750
   JobID=25260750 Status=Running; MinorStatus=Input Data Resolution; 
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

   $ dirac-wms-job-status  25260750
   JobID=25260750 Status=Done; MinorStatus=Execution Complete; 
   Site=LCG.UKI-NORTHGRID-MAN-HEP.uk;

Get Output Data RASCIL job
---------------------------

**Details at:**  `Simple_Job <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__

.. code:: python

   Note: the RASCIL job has 3 image outputs, so we specify them in 
   OutputSandbox and we take the data locally using command

   $ dirac-wms-job-get-output  25260750
   Job output sandbox retrieved in 
   /raid/scratch/<your-user>/dirac_ui/tests/rascilTests/25260750/
   $ cd 25260750
   $ ls
   imaging_dirty.fits  imaging_psf.fits  imaging_restored.fits  StdOut
   $ cat StdOut   #or StdErr


Job submission - submit .jdl vs .py
------------------------------------

**Job submission - submit .jdl**

-  create .jdl and .sh files

   .. code:: python


      cat simpleR1.jdl
      JobName = "InputAndOuputSandbox";
      Executable = "testR1.sh";
      StdOutput = "StdOut";
      StdError = "StdErr";
      InputSandbox = {"testR1.sh"};
      InputData = {"LFN:/skatelescope.eu/user/c/cimpan/rascil/RASCIL-full1.img"};
      OutputSandbox = {"StdOut","StdErr"};
      OutputData={"imaging_dirty.fits","imaging_psf.fits","imaging_restored.fits"};
      OutputSE ="UKI-NORTHGRID-MAN-HEP-disk";
      Site = "LCG.UKI-NORTHGRID-MAN-HEP.uk";


      cat testR1.sh
      #!/bin/bash
      singularity exec --cleanenv -H $PWD:/srv --pwd /srv -C RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py;

-  Submit the job

   .. code:: python


      bash-4.2$ dirac-wms-job-submit simpleR1.jdl
      JobID = 25260750

      bash-4.2$ dirac-wms-job-status 25260750
      JobID=25260750 Status=Running; MinorStatus=Input Data Resolution; 
      Site=LCG.UKINORTHGRID-MAN-HEP.uk;

      bash-4.2$ dirac-wms-job-status 25260750
      JobID=25260750 Status=Done; MinorStatus=Execution Complete; 
      Site=LCG.UKINORTHGRID-MAN-HEP.uk;

-  Get output data and output file

   .. code:: python


      bash-4.2$ dirac-wms-job-get-output-data 25336768
      Job 25336768 output data retrieved
      bash-4.2$ ls
      -rw-r--r--. 1 <your-user> users6 2102400 May 14 17:32 imaging_dirty.fits
      -rw-r--r--. 1 <your-user> users6 2102400 May 14 17:32 imaging_psf.fits
      -rw-r--r--. 1 <your-user> users6 2102400 May 14 17:32 imaging_restored.fits

      bash-4.2$ dirac-wms-job-get-output 25336768
      Job output sandbox retrieved in
      /raid/scratch/<your-user>/dirac_ui/tests/rascilTests/ 25336768/
      bash-4.2$ cd 25336768
      bash-4.2$ ls
      StdErr StdOut
      bash-4.2$ cat StdErr
      INFO: Convert SIF file to sandbox...
      INFO: Cleaning up image...

**Job submission - submit .py**


-  Set up environment variables:

   .. code:: python

         
      #SET THE PATH PYTHON 2.7 INTO $PATH
      #PATH to python 2.7 added
      eg bash-4.2$ export PATH=/usr/local/casa/bin/python:$PATH

-  the job to be submitted and the .sh script

   .. code:: python


      bash-4.2$ cat jobpy.py
      import os
      import sys
      import time
      # setup DIRAC
      from DIRAC.Core.Base import Script
      Script.parseCommandLine(ignoreErrors=False)
      from DIRAC.Interfaces.API.Job import Job
      from DIRAC.Interfaces.API.Dirac import Dirac
      from DIRAC.Core.Security.ProxyInfo import getProxyInfo
      SitesList = ['LCG.UKI-NORTHGRID-MAN-HEP.uk']
      SEList = ['UKI-NORTHGRID-MAN-HEP-disk']
      dirac = Dirac()
      j = Job(stdout='StdOut', stderr='StdErr')
      j.setName('TestJob')
      j.setInputSandbox(["testR1py.sh"])
      j.setInputData(['LFN:/skatelescope.eu/user/c/cimpan/rascil/RASCIL-full1.img'])
      j.setOutputSandbox(['StdOut','StdErr'])
      j.setOutputData(['imaging_dirty.fits','imaging_psf.fits','imaging_restored.fits'],
      outputSE='UKI-NORTHGRID-MAN-HEP-disk')
      j.setExecutable('testR1py.sh')
      jobID = dirac.submitJob(j)
      print 'Submission Result: ', jobID


      bash-4.2$ cat testR1py.sh
      #!/bin/bash
      singularity exec --cleanenv -H $PWD:/srv --pwd /srv -C RASCIL-full1.img python3 /rascil/examples/scripts/imaging.py

-  Submitting the job

   .. code:: python


      bash-4.2$ python jobpy.py
      Submission Result: {'requireProxyUpload': False, 'OK': True, 'rpcStub':
      (('WorkloadManagement/JobManag er', {'delegatedDN':
      None, 'timeout': 600, 'skipCACheck': False, 'keepAliveLapse': 150,
      'delegatedGroup ': None}), 'submitJob', ('[ \n
      Origin = DIRAC;\n Executable = "$DIRACROOT/scripts/dirac-jobexec";
      \n StdError = StdErr;\n LogLevel = info;\n OutputSE = UKI-NORTHGRIDMAN-
      HEP-disk;\n InputSa ndbox = \n {\n
      "testR1py.sh",\n "SB:GridPPSandboxSE|/SandBox/i/iulia.c.cim
      pan.skatelescope.eu_user/cf8/ca6/cf8ca689995e24c01c068eb6f34126b8.tar.bz2"\n
      };\n JobName = T estJob;\n Priority = 1;\n
      Arguments = "jobDescription.xml -o LogLevel=info";\n JobGroup = skat
      elescope.eu;\n OutputSandbox = \n {\n StdOut,\n
      StdErr,\n Sc ript1_testR1py.sh.log\n
      };\n StdOutput = StdOut;\n InputData = LFN:/skatelescope.eu/user/c
      /<your-user>/rascil/RASCIL-full1.img;\n JobType = User;\n OutputData = \n
      {\n imagin g_dirty.fits,\n
      imaging_psf.fits,\n imaging_restored.fits\n };\n]',)), 'Va
      lue': 25344748, 'JobID': 25344748}

-  Get the results

   .. code:: python


      bash-4.2$ dirac-wms-job-get-output 25344748
      Job output sandbox retrieved in 
      /raid/scratch/<your-user>/dirac_ui/tests/rascilTests/25344748/

      bash-4.2$ cd 25344748
      bash-4.2$ ls
      Script1_testR1py.sh.log StdOut

      bash-4.2$ dirac-wms-job-get-output-data 25344748
      Job 25344748 output data retrieved
      bash-4.2$ ls
      imaging_dirty.fits imaging_psf.fits imaging_restored.fits
      Script1_testR1py.sh.log StdOut
      
Useful Links
-------------

-   `GridPP_user-guide <https://github.com/GridPP/user-guides>`__

-   `Getting_on_the_grid <https://github.com/gridpp/user-guides/tree/master/getting-on-the-grid>`__

-   `Grid_user_crash_course <https://www.gridpp.ac.uk/wiki/Grid_user_crash_course>`__

-   `Quick_Guide_to_Dirac <https://www.gridpp.ac.uk/wiki/Quick_Guide_to_Dirac>`__

-   `Getting_started_User_Jobs <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/index.html>`__

-   `Getting_started_Data_Management <https://dirac.readthedocs.io/en/latest/UserGuide/CommandReference/DataManagement/index.html>`__

-   `Getting_started_Command_Line <https://dirac.readthedocs.io/en/latest/UserGuide/GettingStarted/UserJobs/CommandLine/index.html>`__


