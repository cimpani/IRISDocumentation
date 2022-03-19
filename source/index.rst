.. IRIS documentation master file, created by
   sphinx-quickstart on Wed Feb 10 18:12:17 2021.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` direct
   
   
Welcome to the documentation for IRIS Radio Astronomy
=====================================================

IRIS is a cooperative community creating digital research infrastructure to support STFC science. This
site documents how to sign up for `IRIS resources <https://irisdocumentation.readthedocs.io/en/latest/signupIR.html>`__  like IRIS (through certificate), Galahad, DIRAC/Safe, how to submit jobs on these resources and provides example Radio Astronomy workflows.

For more details about IRIS, go to `What_is_IRIS <https://www.iris.ac.uk/>`__

**IRIS grid nodes** in Manchester are 16 core CPU 1.5 TB RAM (13 machines) and 40 core CPU 3 TB RAM (one machine), and no GPU
**Galahad** 16 core CPU 1.5 TB RAM 2x A100 GPU machines are funded by IRIS, 17 machines. 

.. note::

  Recommendation to new starters to read through and follow steps: 
#. `Signing up for IRIS Resources <https://irisdocumentation.readthedocs.io/en/latest/signupIR.html>`_ for the resource of your choice: IRIS (through certificate), Galahad or DIRAC/Safe.
#.  have a quick read through  `Before starting: useful tools, basic knowledge <https://irisdocumentation.readthedocs.io/en/latest/cambridgehpc.html>`_ and `Singularity: basic usage and containers <https://irisdocumentation.readthedocs.io/en/latest/singularity.html>`_
#. `Job submission on IRIS Resources - Galahad, Dirac SAFE and IRIS(cert) <https://irisdocumentation.readthedocs.io/en/latest/JobSub.html>`_
#. `Benchmarking processing performance and Parameterized jobs IRIS(cert) <https://irisdocumentation.readthedocs.io/en/latest/BENCHM.html>`_ it is about monitoring job performance on IRIS  grid nodes, using .jdl jobs. PrMON can also be used on slurm jobs on Galahad.


.. note::

   This project is under construction.


.. toctree::
   :maxdepth: 3
   :caption: Contents:

   signupIR
   cambridgehpc
   singularity
   JobSub
   BENCHM
   rascilUC
