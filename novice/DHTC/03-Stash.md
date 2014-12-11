---
layout: lesson
root: ../..
title: Data storage and transfer
---
<div class="objectives" markdown="1">

#### Objectives
*   Discover how to transfer input and output data  
*   Learn the what, why, how, and whens of using Stash
</div>


<h2> Overview </h2>
In this lesson, we will learn the basics of data storage and transfer on OSG Connect. 

<h2> Stash </h2>
OSG Connect provides a storage system called Stash.  Stash provides a place to
store data needed for jobs or output from jobs in the medium term.  Since
*stash* is not backed up, you should transfer job outputs from stash to your
local system as soon as practical.

This lesson will go over accessing *stash* using the OSG Connect login node as
well as other methods such as Globus, and HTTP.
<h2>Exploring the Stash system</h2>

First, we'll look at accessing *Stash* from the login node. You'll need to log
in to OSG Connect:
~~~
ssh username@login.osgconnect.net #Connect to the login node with your username
passwd:       # your password
~~~

Once done, you can change to the 'stash' directory in your home area:
~~~
$ cd ~/stash    # This is your *stash* directory
~~~
{:class="in"}

This directory is an area on *stash* that you can use to store files and
directories.  It functions just like any other UNIX directory although it has
additional functions that we'll go over shortly.

For future use, let's create a file in *stash*:
~~~
$ cd ~/stash
$ echo "Hello world" > my_hello_world
~~~


<h2> File Transfer to Stash </h2> 

We can transfer files to stash using scp or Globus. First, let's 
look at transferring files using scp.  Scp allows you to transfer files and
directories from your OSG Connect account using your ssh credentials.  It works
with any unix file or directory is not specific to *stash*.

To transfer a file from *stash* using scp, you'll need to run scp with the
source and destination.  Files on remote systems are indicated using
user@machine:/path/to/file .  Let's copy the file we just created from stash to
our local system:

~~~
$ scp username@login.osgconnect.net:~/data/my_hello_world .
~~~

As you can see, scp uses similar syntax to the cp command that you were shown
previously.  To copy directories using scp, you'll just pass the -r option to
scp.  E.g:

 ~~
$ scp -r username@login.osgconnect.net:~/data/my-directory .
~~~

As an exercise, create a directory with a file called hello_world_2 in the
~/data directory and copy it from stash to your local system.  Then create a
directory called hello_world_3 on your local system and copy it to the data
directory.

{:class="in"}

<h2> Downloading data from Stash </h2> 

Let us do an example calculation to understand the use of *stash* and how we download 
the data from the web. We will peform 
molecular dynamics simulation of a small protein in implicit water. To get the
necessary files, we use the *tutorial* command on OSG. 

Log in OSG

~~~
$ ssh username@login.osgconnect.net
~~~

type 

~~~
$ tutorial stash-namd
$ cd ~/osg-stash-namd
~~~

~~~
namd_stash_run.submit #Condor job submission script file.
namd_stash_run.sh #Job execution script file.
ubq_gbis_eq.conf #Input configuration for NAMD.
ubq.pdb #Input pdb file for NAMD.
ubq.psf #Input file for NAMD.
par_all27_prot_lipid.inp #Parameter file for NAMD.
~~~

The file - "par_all27_prot_lipid.inp" is the parameter file and is required for 
the NAMD simulations. The parameter file is common data file for the NAMD
simulations. It is a good practice to keep the common files, like  the parameter file 
in our example, in the *stash* storage.  

~~~
mv par_all27_prot_lipid.inp ~/data/public/.  #moving the parameter file from the local directory ~/osg-namd-stash  to ~/data/public 
~~~

You can view the parameter file appear on WWW

~~~
http://stash.osgconnect.net/+yourusername
~~~

Now we want the parameter file available on the execution (worker) machine when the 
simulation starts to run. As mentioned early, the data on the *stash* is available to 
the execution machines. This means the execution machine can transfer the data from 
*stash* as a part of the job execution. So we have to script this in the job execution 
script. 

You can see that the job execution script "namd_stash_run.sh" has the following lines:

~~~
#!/bin/bash  
source /cvmfs/oasis.opensciencegrid.org/osg/modules/lmod/5.6.2/init/bash #sourcing a shell specific file that adds the module command to your environment
module load namd/2.9  #loading the namd module
wget --no-check-certificate http://stash.osgconnect.net/+username/par_all27_prot_lipid.inp # Getting the data from the *stash*. Insert your username. 
namd2 ubq_gbis_eq.conf  #Executing the NAMD simulation
~~~

In the above script, you will have to insert your "username" in URL address. The
parameter file located on *stash* is downloaded using the #wget# utility.  
 

Now we submit the NAMD job. 

~~~
$ condor_submit namd_stash_run.submit 
~~~

Once the job completes, you will see non-empty "job.out.0" file where 
the standout output from the programs are written as default.   

~~~
$ tail job.out.0

WallClock: 6.084453  CPUTime: 6.084453  Memory: 53.500000 MB
Program finished.
~~~

The above lines indicate the NAMD simulation was successful. 

 
<div class="keypoints" markdown="1">

#### Key Points
* Data on *stash* is quickly accessed by the worker machines. 
* *stash* is located at ~/data on login.osgconnect.net. 
* *scp* and *rsync* are useful tools for data transfer.
</div>

