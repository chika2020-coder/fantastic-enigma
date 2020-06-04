                     clonEasy README FILE
                      version 2005-02-26
                  (text revised on 2011-03-15)
                 by Josep Sempau & Andreu Badal
              Universitat Politecnica de Catalunya


CONTENTS:

0. COPYRIGHT AND DISCLAIMER
1. PURPOSE OF THIS SOFTWARE PACKAGE
2. WHAT YOU GET
3. HOW TO USE IT
4. WHERE DO I GET THE LATEST UPDATE?
5. SOME REMARKS
6. WHO DO I COMPLAIN TO?
7. PREFERRED CITATION


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
0. COPYRIGHT AND DISCLAIMER

cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc
c                                                                      c
c  clonEasy                                                            c
c  Copyright (c) 2004                                                  c
c  Universitat Politecnica de Catalunya                                c
c                                                                      c
c  Permission to use, copy, modify and re-distribute unaltered copies  c
c  of this software package and its documentation for any purpose is   c
c  hereby granted without fee, provided that this copyright notice     c
c  appears in all copies. The Universitat Politecnica de Catalunya     c
c  makes no representations about the suitability of this software for c
c  any purpose. It is provided "as is" without express or implied      c
c  warranty.                                                           c
c                                                                      c
cccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccccc


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
1. PURPOSE OF THIS SOFTWARE PACKAGE

clonEasy is a collection of Linux scripts that implement ssh-based communication between a "master" computer and a set of "clones". The purpose of this communication is to execute a code that performs a Monte Carlo simulation on all the clones at the same time. The code is unique but each clone is fed with a different set of random seeds so that clonEasy is effectively an implementation of a parallel calculation system. Any ssh-accessible computer connected to the Internet that uses a bash-style shell (Linux and Unix systems) on which you have an account can become one of your clones.

Throughout this document it is assumed that users know how to operate a Linux system and how to write basic Linux scripts for the bash and sh shell (clonEasy does not work properly on other shells such as csh or bsh). The term "Linux" is used because we have tested our package only on machines running this operating system (more precisely, on the SuSE, Red Hat and Debian distributions). More than likely, it will work equally well on any Unix flavor with the bash shell. Actually, it would even run on Windows if some minor modifications were introduced in the scripts (such as '$1' -> '%1' and so on) and if the Windows computers had the equivalent of an ssh client/server program installed and running.

We also assume that users are experienced Monte Carlo (MC) simulators. Terms such as random seeds, main program, make file, g77 compiler, etc. are used without further explanation.

The clonEasy package is free and open software. Please see section 7 if you want to know how to cite this work.

We hope you enjoy it.


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
2. WHAT YOU GET

The user interface is formed by the following script files:

  clon-setup     : sets up the environment for clonEasy to work.
  clon-upload    : uploads files from the master to the clones.
  clon-make      : executes the compilation script on each clone.
  clon-run       : uploads the seeds and executes a program on all clones.
  clon-download  : downloads files from the clones to the master.
  clon-remove    : removes files and directories from all clones.

When no arguments are given, some of these scripts issue an informative message to the screen explaining their usage.

The package also includes the following files:

  README.txt       : this file
  clon.sample.tab  : a sample clone table; copy and modify it to fit
                     your configuration.
  combine.sample.in: a sample input file for the combine.f code.
  make.sample      : an example of a simple compilation script.
  clon-run_aux     : a script that is used internally; do not execute it.
  replicate.f      : a FORTRAN 77 code used internally.
  combine.f        : a FORTRAN 77 program that can be used to post-process
                     the output files obtained from the clones.


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
3. HOW TO USE IT

A) Install (to be performed only once)

A.1 Prepare the master

- Create a working directory for clonEasy. We shall call it WD from now on. Copy clonEasy.zip to the WD and unzip.

- Make all scripts executable, e.g., by executing
    $ chmod u+x clon-*
from the WD.

In addition, make sure that the script clon-run_aux (see above) has write ('w') permission,
    $ chmod u+w clon-run_aux

- Edit the script clon-setup and replace '<write_your_WD_here>' by your actual WD. This script sets up the environment for clonEasy to work and must be run each time you start a new session on the master. To run, execute
    $ source clon-setup
Do not forget the word 'source'. (It works in the bash and sh shells; it  may not work properly on other Unix shells.)

Alternatively, you may define a variable CLONWD and change the PATH of the shell by modifying your login script (for instance .bashrc if you are using this shell) adding the two commands that are contained in clon-setup. This will save you the trouble of having to introduce the above command each time you log-in.

- Compile and link the FORTRAN 77 code 'replicate.f', e.g. with
    $ g77 replicate.f -o replicate.x -O
Make sure the output is named 'replicate.x' and that you have execution rights on it.

- Compile and link the FORTRAN 77 code 'combine.f', e.g.
    $ g77 combine.f -o combine.x -O


A.2 Prepare the clones

- Check that all clones have umasks that are (rwx------) at least, e.g., by including
    $ umask 077
(or any other three-digit value starting with '0') on the login script of each clone.

- Grant ssh access to clones from the master using public key authentication. In short, this is done by executing the command ssh-keygen on your master. This will create two files, one with a name that ends '.pub', which contains the public key. The other file contains the private key that matches the public key. Take the file with the public key and append it to the file authorized_keys2 (or something similar) found in the directory $HOME/.ssh/ of each clone. The private key file must be moved to the .ssh/ directory of the master. See man ssh for more details on how to carry out this step and for the precise file names utilized in your Linux/Unix system.

To make sure this step has been done properly, check that clones do not request a password when you log-in (this is what "public key authentication" means).

- Identify the compiler used in each clone and its optimization options.


(The next steps describe actions to be taken each time a job is to be run. They imply changes only on the master files.)


B) Upload the job

- Create a directory for your job. This will be called JD hereafter. Put all the files necessary to run your case in it.

- Your main program has to be written in such a way that it reads the initial random seeds from an external file named 'rngseed.in' located in the program's current directory. This is essential to enable different clones to be initialized with different seeds automatically.

- Create a clone table file following the example clon.sample.tab provided in the distribution. See this file for details about its structure. Put it in the JD.

- Run clon-upload to copy the files necessary for the simulation to each clone.


C) Compile the job

- Clones with CPUs or architectures different from that of the master may require the MC code to be compiled locally (i.e. on each clone). For this reason, clonEasy allows the execution of a different make-file in each clone. The file 'make.sample' provided with clonEasy is an example of a simple make-file script. There should be a make-file for each clone or group of clones using the same compiler. You may create an empty make-file for those clones that will run the executable file uploaded from the master and therefore do not need local compilation.

The file name of the make-file to be executed on each clone is defined by the user in the clone table file (see above).

- If you did not upload the make file in step B, do it now.

- Run clon-make to execute, on each clone, the make-file that is specified in the clone table file (only if compilation on clones is needed).

Note: If your MC code uses some non-standard routines (e.g. time routines with FORTRAN 77), make sure they work on all clones. Otherwise, avoid their use.


D) Run the job

- Run clon-run. This script creates a file 'rngseed.in' for each set of random seeds listed in the clone table file and uploads it to the corresponding clone. Afterwards, it executes the command specified in the command line by the user (intended to run his/her MC job) when invoking clon-run.

clon-run accepts an optional last argument intended to allow the user to specify a job name, say 'job09', in order to redirect the program standard input and output. In the case that this job name is provided, it is assumed that the input and output files to be used to run the MC simulation are named, e.g. 'job09.in' and 'job09.out'. So, the job should be able to be run like this:
    $ MCcode.x < job09.in > job09.out

Note: Due to some peculiarities of ssh, this step leaves a set of sleeping ssh processes on the master and also, perhaps, on the clones. They should die when the job finishes. Otherwise, use the kill command to terminate them manually. Apparently, this latter action does NOT terminate the execution of the job itself, so you can kill the ssh processes even before your job is done.


E) Job control

- If you intend to implement some kind of job control during execution, this is best carried out by introducing "commands" into an external file (say 'command.in') which is read by your main program at execution time. For instance, the possible commands can be coded as integer numbers that tell the program things such as "stop now" or "write your results up to this point".

- Assuming the preceding job control scheme has been implemented in your main program, clone job control is performed by simply uploading 'command.in' from the master into the clones using clon-upload.


F) Download results

- Run clon-download to get the output files from all the clones. Notice that all clones use the same file name (e.g. job.out). Therefore, to avoid overwriting it, they are copied on the master disk with a prefix formed by the clone's nickname (as defined in the clone table file) and a dot, as in clon1.job.out, clon2.job.out, etc.


G) Process results

- Since only you know what your main program does and how it writes its results, only you can write a program that specifically reads the output files and process them to obtain the overall average, uncertainty, number of histories, simulation time, etc.

However, if some simple formatting rules are observed for the output files, the auxiliary code 'combine.f' can be employed to read and process them. After compilation (see above), this tool is executed with
    $ combine.x < combine.in > program.out

The overall results of your simulation case are now contained in program.out. Edit the file combine.sample.in and read the notes therein for details on how to format your output files and on how to create a valid input file (i.e. combine.in in the example above).


H) Clean up

- Run clon-remove to remove a file from all clones or to remove an entire directory structure. USE WITH CARE!!


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
4. WHERE DO I GET THE LATEST UPDATE?

clonEasy can be freely downloaded from (please note that the following address may be case sensitive)

http://www.upc.es/inte/downloads/clonEasy.htm



>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
5. SOME FINAL REMARKS

- Every script records the Linux command that called it on a file named log.txt. In addition, almost all scripts create intermediate temporary script files (named *.tmp) on the current directory that contain commands executed internally. Temporary scripts are not deleted upon completion. Thus, users can inspect what exactly has been done after executing a script. If you are not interested, just remove them at any time (but not while a clonEasy script is still running) with
    $ rm -f *.tmp

- When replicate.x encounters an error in the execution of the scripts (for example an incomplete clone table file), clonEasy is stopped. In this situation, an error message is displayed on the screen and saved in a file called error.log.

- In the clones, job files are copied into and the code run from a subdirectory called as the clone's nickname. This subdirectory is located below the clone's working directory that is entered by the user as a parameter in the execution of the scripts. This allows the multiple execution of the same job on a multiprocessor computer (by specifying the same machine name but different nicknames and random seeds on the clone table file) or the use of clusters with clones that share the same disk. A dot can be entered to specify that the working directory is the HOME directory of each clone.

- Some of the scripts mentioned above may take a sizeable amount of time to complete. In particular, clon-make has to compile and link all programs on all clones. Communication over the Internet can be slow too. If any connection is broken (e.g. because lines are down), it will take some time for ssh to realize that this has happened and it may seem that it has hung up. Be patient and wait until the scripts finish.


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
6. WHO DO I COMPLAIN TO?

Report bugs, problems, praises and complaints to josep.sempau@upc.es.


>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>
7. PREFERRED CITATION

If you find penEasy useful, please cite the following reference (given in BibTeX format for your convenience) in your publications:

@ARTICLE{badal2006,
  AUTHOR  ={A. Badal and J. Sempau},
  TITLE   ={{A package of Linux scripts for the parallelization of Monte Carlo simulations}},
  JOURNAL ={Comput. Phys. Commun.},
  Volume  ={175},
  YEAR    ={2006},
  Pages   ={440 -- 450}
}

The paper in PDF format can be freely downloaded from:
  http://www.upc.es/inte/downloads/clonEasy.htm


>>>> EOF >>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>>

