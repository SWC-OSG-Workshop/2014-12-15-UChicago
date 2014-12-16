---
layout: lesson
root: ../..
title: Scaling up compute resources 
---
<div class="objectives" markdown="1">

#### Objectives
*   Learn why to scale up the computational resources.
*   Learn how to scaling up the computational resources  
</div>

## Overview

Scaling up the computational resources is a big advantage for doing     
certain large scale calculations on OSG. Consider the extensive         
sampling for a multi-dimensional Monte Carlo integration or molecular   
dynamics simulation with several initial conditions. These type of      
calculations require submitting lot of jobs.                  

In the previous example, we submitted the job to a single worker
machine. About a million CPU hours per day are available to OSG users
on an opportunistic basis.  Learning how to scale up and control large
numbers of jobs to realizing the full potential of distributed high
throughput computing on the OSG.

In this section, we will see how to scale up the calculations with
simple example. Once we understand the basic HTCondor script, it is easy
to scale up.

We'll begin with the same tutorial as we used in the previous chapter.  If
you weren't with us then:

~~~
$ tutorial quickstart
$ cd quickstart
~~~

As we discussed in the previous section on HTCondor scripts, we need to 
prepare the job execution and the job submission scripts. Here again
is our job execution script:

## Submitting jobs concurrently

What do we need to do to submit several jobs simultaneously? In the
first example, Condor returned three files: out, error, and log. If we
want to submit several jobs, we need to track these three files for each
job. An easy way to do this is to add the $(Cluster) and $(Process)
macros to the HTCondor submit file. Since this can make our working
directory really messy with a large number of jobs, let's tell HTCondor
to put the files in a directory called log. Here's what the second (less
verbose) submit file looks like -- `tutorial02.submit`:

~~~
Universe = vanilla

Executable = short.sh 

Error = log/job.error.$(Cluster)-$(Process)     # differs from previous script
Output = log/job.output.$(Cluster)-$(Process)   # differs from previous script
Log = log/job.log.$(Cluster)                    # differs from previous script

Queue 25                                        # differs from previous script
~~~

Note the `Queue 25`.  This tells Condor to enqueue 25 copies of this job
as one cluster.  Before submitting this cluster, we should make sure the
log directory exists.

~~~
$ mkdir log
~~~

You'll see something like the following upon submission:

~~~
$ condor_submit tutorial02.submit
Submitting job(s).........................
25 job(s) submitted to cluster 837.
~~~

Apply your `condor_q` and `connect watch` knowledge to see this job
progress.  Once this cluster completes, take a look in the `log/`
directory.

~~~
$ ls log
0 job.error.837-0   0 job.error.837-20  0 job.output.837-0   0 job.output.837-20
0 job.error.837-1   0 job.error.837-21  0 job.output.837-1   0 job.output.837-21
0 job.error.837-10  0 job.error.837-22  0 job.output.837-10  0 job.output.837-22
0 job.error.837-11  0 job.error.837-23  0 job.output.837-11  0 job.output.837-23
0 job.error.837-12  0 job.error.837-24  0 job.output.837-12  0 job.output.837-24
0 job.error.837-13  0 job.error.837-3   0 job.output.837-13  0 job.output.837-3
0 job.error.837-14  0 job.error.837-4   0 job.output.837-14  0 job.output.837-4
0 job.error.837-15  0 job.error.837-5   0 job.output.837-15  0 job.output.837-5
0 job.error.837-16  0 job.error.837-6   0 job.output.837-16  0 job.output.837-6
0 job.error.837-17  0 job.error.837-7   0 job.output.837-17  0 job.output.837-7
0 job.error.837-18  0 job.error.837-8   0 job.output.837-18  0 job.output.837-8
0 job.error.837-19  0 job.error.837-9   0 job.output.837-19  0 job.output.837-9
0 job.error.837-2   4 job.log.837       0 job.output.837-2
~~~

You see there's one error file and one output file for each job, plus
the `job.log.837` file for tracking the whole cluster.


## Interlude: utilization plots

Before we continue, let's look at a URL: [your OSG Connect home
page](http://osgconnect.net/home).  If you have not signed in, you'll be
redirected back to the main site.  Sign In as you did the first time you
signed up, and then click again on the [your OSG Connect home link]
(http://osgconnect.net/home).

You see a number of graphs and plots here showing things happening
in OSG Connect.  We'll go over these briefly, then return later.


## Passing arguments to executables 

Sometimes it's useful to pass arguments to your executable from your
submit file. For example, you might want to use the same job script
for more than one run, varying only the parameters. You can do that
by adding `Arguments` to your submission file. Let's try that with
`tutorial03.submit`.

We want to run many more instances for this example: 100 instead of only
10. To ensure that we don't collectively overwhelm the scheduler let's
also dial down our sleep time from 15 seconds to 5.

Submission script:
~~~
Universe = vanilla 

Executable = short.sh 
Arguments = 5 # to sleep 5 seconds 

Error = log/job.err.$(Cluster)-$(Process) 
Output = log/job.out.$(Cluster)-$(Process) 
Log = log/job.log.$(Cluster) 

Queue 100
~~~


Let's take a look again at our `short.sh` script:

~~~
#!/bin/bash
# short.sh: a short discovery job
printf "Start time: "; /bin/date
printf "Job is running on node: "; /bin/hostname
printf "Job running as user: "; /usr/bin/id
printf "Job is running in directory: "; /bin/pwd
echo
echo "Working hard..."
sleep ${1-15}
echo "Science complete!"
~~~

Look at the `sleep` line near the end.  This is the same script as we
used before, but we overlooked this peculiar syntax.  `${1-15}` means to
insert the value `15`, _unless_ there is a `$1` supplied to the script.
Recall from the lessons on shell that `$1` means the first argument --
so what does this mean?  It means that if we give the `short.sh` script
an argument, that's how many seconds it sleeps for.  If we don't, it
sleeps for 15 seconds.  And in our job submission script, we added
`Arguments = 5`, so now our workers will sleep for five seconds.

Let's submit:
~~~
$ condor_submit tutorial03
Submitting job(s)..........
100 job(s) submitted to cluster 938. 
~~~

This will take a little longer, of course, but we're still hardly doing
any real work, so it shouldn't be too long.

~~~
$ condor_q username | tail -1
100 jobs; 0 completed, 0 removed, 100 idle, 0 running, 0 held, 0 suspended
~~~

Very good, we're waiting on 100 jobs.  Let's use `connect watch` to
watch for job completions.  As soon as you see some jobs enter R state
(running), press control-C, and let's introduce a new command:

~~~
$ connect histogram
Val                               |Ct (Pct)    Histogram
unl.edu                           |46 (68.66%) ████████████████████████████████▏
bu.edu                            |13 (19.40%) █████████▏
uconn.edu                         |2 (2.99%)   █▌
CRUSH-OSG-10-5-220-34             |1 (1.49%)   ▊
ufhpc                             |1 (1.49%)   ▊
LAW-D-SBA01-S2-its-c6-osg-20141013|1 (1.49%)   ▊
CRUSH-OSG-10-5-10-33              |1 (1.49%)   ▊
iu.edu                            |1 (1.49%)   ▊
vt.edu                            |1 (1.49%)   ▊
~~~

This command gives us a simple histogram of where on the grid our jobs
are running.  The column on the left is (for the most) a list of _sites_
that OSG jobs run on.  At times we don't correctly group job locations
together. For example, the two rows for CRUSH-* above are really the
same site, but histogram doesn't know about that site (yet) so it
displays as two.  But most of the big sites are mapped correctly.  You
see that in my case, 67 of my 100 jobs have begun running, and among
them 69% (46 of 67) are running at University of Nebraska at Lincoln.

`connect histogram` gives metrics on current jobs.  As jobs complete,
they no longer appear.  How to see where jobs have already run? `connect
histogram --last` shows the run sites of your *last* job cluster.

~~~
$ connect histogram --last
Val                               |Ct (Pct)    Histogram
uc3                               |49 (49.00%) ████████████████████████████████▏
bu.edu                            |21 (21.00%) █████████████▊
uconn.edu                         |11 (11.00%) ███████▎
unl.edu                           |9 (9.00%)   ██████
mwt2.org                          |3 (3.00%)   ██
c5a-s22.ufhpc                     |3 (3.00%)   ██
LCS-215-021-S2-its-c6-osg-20141013|3 (3.00%)   ██
cinvestav.mx                      |1 (1.00%)   ▊
~~~

Now that we've run some much larger jobs, let's revist [your OSG Connect
home page](http://osgconnect.net/home).  Reload the page, and observe
what happens in the second graph.  You can click the **large** link to
blow this up to full size.  You will see your recent jobs occupying
columns to the right edge of the plot -- this is another place you can
monitor to see how your work is progressing.

<div class="keypoints" markdown="1">

#### Key Points
*    Scaling up the computational resources on OSG is crucial to taking full advantage of grid computing.
*    Changing the value of *Queue* allows the user to scale up the resources.   
*    *Arguments* allows you to pass parameters to a job script.
*    $(Cluster) and $(Process) can be used to name log files uniquely.
*    `connect histogram` gives a nice plot of resource assignments.
*    Your OSG Connect home page gives other useful plots.
</div>
