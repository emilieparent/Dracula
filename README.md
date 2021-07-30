# DRACULA - Determining the Rotation Count of Pulsars
A pulsar phase connection method

Code written by Paulo Freire. 

Paper with description of concepts is now online: https://arxiv.org/abs/1802.07211

Updates and instructions by Paulo Freire, based on initial description by Erik Madsen.

Major updates

- Oct. 10. 2020: The automatic version of sieve.sh, dracula.sh !

- Oct. 23: new version of dracula.sh that does far fewer sorts when we have longs lists of partial solutions, saves results of processing in the occasions when it sorts, and e-mails user when it finds a solution.

- Oct. 29: new version that lets user know about new solution immediately after it is computed, not later when its chi2 is sorted.

- Oct. 31: improved description, functionality, added "usage" below.

- Jan. 19 2021: changed gap names from JUMPX to GAPX. This allows automatic building of list of gap names from TOA list, without needing to specify it independently in the script.

- March 2 2021: Added list of published pulsars that have been connected using this software (see at the end).

- July 29 2021: By popular request, I posted TEMPO patch that allows it to print very large values of reduced chi2 (see below in 'known issues'). IMPORTANT: YOU NEED TO DO THIS CHANGE IN ORDER FOR THE EXAMPLE BELOW TO WORK!

### Instructions (which assume familiarity with TEMPO)

Preparation:

You should have an initial ephemeris (parfile) and set of TOAs (timfile). Place JUMPs around every epoch (each comprising of a group of TOAs) except one. If your initial parfile is reasonable, you should be able to run TEMPO on this and get pretty flat residuals.

Beware of groups of TOAs close to rotational phase 0.5, some of those can appear at rotational phase -0.5. In that case TEMPO is assuming the wrong rotation count, whenever it happens it cannot converge on an accurate solution. This can be fixed by using PHASE +1 or PHASE -1 statements, as I do in the example timfiles.

If necessary, put an EFAC in your timfile such that this step also results in a reduced chi-squared (henceforce "chi2") of ~1.

Epochs can be joined together by removing JUMPs from the timfile. Try doing this between nearby epochs, by removing two successive JUMP statements and inserting a "PHASE N" (where N is some integer number of phase wraps) between them. Some value of N (maybe 0) will hopefully result in a chi2 ~1. If changing N by +/-1 should give a chi2 that is considerably larger than 1, then you have an unambiguous connection, i.e., you connected that gap. In this case, move to another gap where you feel you can now get an unambiguous solution.

If your dataset allows it, then you can proceed like this until all TOAs are connected. In that case, you have determined the full phase connection for the pulsar!

If, on the other hand, during the phase connection effort you reach a stage where, for all unconnected gaps between (connected) TOA sets, you have multiple values on N giving acceptable fits, you have only ambiguous gaps left: in this case you cannot proceed with manual connection. Then you need to use one of the scripts below.

******* First script: sieve.sh *******

This is an earlier, now obsolete version of the phase connecting script. This is mostly here for reference, since this was described and used by Freire & Ridolfi (2018). Also, some of the details are important for understanding how dracula.sh works.

Edit sieve.sh. First, enter your TEMPO, basedir, rundir, timfile, and parfile information at the top of the file. Then edit with prev_labels ="0" and next_label="A". Also, edit the threshold for an acceptable solution (2.0 is a reasonable number).
Write "PHASEA" in your TOA list where you have the shortest ambiguous gap between TOA groups, also removing the JUMPs around it, like in this example:

...

JUMP


JUMP

7               1390.000 51582.2548632839670   13.657                 0.00000

7               1390.000 51582.3201388983131   25.329                 0.00000

7               1390.000 51582.3850678691313   16.834                 0.00000

C JUMP

PHASEA

C JUMP

7               1390.000 51589.2534739821375   29.849                 0.00000

7               1390.000 51589.3336799053180   28.445                 0.00000

JUMP


JUMP

...

Run the script. This will find all the acceptable integers for the gap tagged with PHASEA. These are written in file WRAPs.dat, which that tabulates the chi2 for each of these combinations. These are then automatically sorted into a new acc_WRAPs.dat file (the starting acc_WRAPs.dat file, generated automatically and consisting of a single 0). This acc_WRAPs.dat file is copied to acc_WRAPs_A.dat as a record.

Now, in the TOA file, include the tag PHASEB in the nest shortest gap, commenting out the JUMPs around it. Then edit sieve.sh, with prev_labels="0 A" and next_label="B". Run sieve.sh again. Every acceptable combination of PHASEA that was in your acc_WRAPs.dat file will be tested along with a range of PHASEB values. These are determined by finding the minimum of the chi2 parabola in each case. The resulting list of acceptable solutions is sorted into a new version of file acc_WRAPs.dat (this is copied automatically to acc_WRAPs_B.dat).

This is an iterative process. For your third run, prev_labels="0 A B" and next_label="C". With each additional run, these will 'increment' (on the fourht run, they will be " 0 A B C" and "D").

You might find that normally early on you have relatively few 'acceptable' solutions, and this might balloon out to thousands upon thousands. That's probably OK. Hopefully after a few rounds (which are of the same order as the number of parameters in your initial solution) the number of solutions will stop growing. If the numbers are millions, you can set the chi2 threshold lower, to (for instance) 1.6 instead of 2.0 just so you don't have to wait all day for this to run, you will suddenly see a sharp decrease in the number of solutions.

You might also find that somewhere along the way you need to start fitting an additional parameter in order to keep getting any acceptable solutions. That's simply an edit of your starting parfile.

******* Second script: dracula.sh *******

Using the previous script is a good idea if the number of possible solutions is a few thousands. If it is millions instead, then you have a problem. 
Also, using the previous script requires some manual operation. 

To do things automatically, you can use instead dracula.sh. To use this, you have to edit the tags of all the gaps between groups of TOAs in advance in your .tim file, or at least a few of them, as I did in file 47TucAA.tim. The syntax here is slightly different than in sieve.sh. The example above is written as:

..

JUMP


JUMP

7               1390.000 51582.2548632839670   13.657                 0.00000

7               1390.000 51582.3201388983131   25.329                 0.00000

7               1390.000 51582.3850678691313   16.834                 0.00000

JUMP

C GAPA

JUMP

7               1390.000 51589.2534739821375   29.849                 0.00000

7               1390.000 51589.3336799053180   28.445                 0.00000

JUMP


JUMP

...


Thus, the gap tags must be commented out, and the JUMPs around it cannot be commented out. Note that the JUMP statements around each GAP statement should be offset by two lines, because that is what the dracula.sh script assumes, so that it can comment them out properly when needed. At the end, you must add a dummy tag, GAP0, also between JUMPs:

...

JUMP

C GAP0

JUMP

This is necessary for the script to start processing the first line in acc_WRAPs.dat. Because of this, all solutions (whether made by sieve.sh or dracula.sh) start with a zero.

After that, enter your TEMPO, basedir, rundir, timfile, parfile information at the top of the script (as in the sieve.sh script) and e-mail, if you want the solutions to be e-mailed to you and not to me. If you're continuing work from sieve.sh, please change the file with the TOAs, as shown above. Then, finally, make it run, by simply calling the script! The timing solution(s) will appear in your base working directory as solution_n.m.par, where n and m are two unique integers that tell you in which cycle was the solution found.

The dracula.sh routine is superior to sieve.sh, and should preferably be used:

- The writing is simpler, more transparent, and overall the script is easier to follow. Part of this is because of the improved logic, and in particular the use of trial.tim as an intermediate file, and the use of gaps.txt as a support file.
- As noted before, it is automatic, very little manual intervention is needed. For each solution, the script not only changes the C GAPX into PHASE +N statements, but it also comments out the JUMP statements around it as required by the partial solution being examined. For this, the use of the intermediate file (trial.tim) is very useful. 
- However, the more important improvement, which is pretty fundamental, is to always prioritize the partial solutions with the lowest chi2, no matter how many gaps they connect. This means that we always get to the timing solution faster (and sometimes _much_ faster) than with sieve.sh, where we must calculate all solutions for each new gap first before moving to the next gap.

Indeed, if you run this script with 47TucAA.tim and 47TucAA.par, you should see the solution emerge in the 46th cycle (i.e., resulting from the processing of the 45th partial solution for which it tries to connect one extra gap), not after more than 400 cycles.

Some notes about the usage of dracula.sh (and some exercises you can do with 47TucAA.tim, to improve familiarity with the script):

- The improvement of dracula relative to sieve.sh was already described in Freire & Ridolfi (2018), in the last paragraph of section 4.3, the delay in the implementation has to do with the fact that only now did a really simple implementation occur to me.
- You don't need to tag all the gaps between TOAs in advance, just enough that you think you might get a unique solution. The file 47TucAA.tim is an example of this. After finding the unique solution for 47 Tuc AA, you can keep connecting manually (i.e., by editing PHASE +N statements for each additional gap betwen TOA groups) in order to verify that the connection is now unambiguous for all the remaining gaps, and chek whether it stays within your chi2 limit or not - and if not, whether fitting any additional parameters helps.
   - NOTE: If you start this manual work with the post-fit solution(s) that appears in $basedir with the name solution_n.m.par, then all previously connected gap tags and JUMP statements around them have to be deleted (or commented out) in the .tim file, because those rotation numbers are already taken into account by solution_n.m.par. One of the advantages of this is that if you set NITS to 1 in that solution, you can see the pre-fit residuals for all TOAs. This gives you a good idea of how good that solution really is at predicting TOAs. Also, starting from this new solution results in much smaller PHASE numbers for all remaining gaps. In the case of 47TucAA.tim, those should all be 0.
- Note that after determining the solution, the script will keep running. This will determine whether the solution reported is unique or not. In the case of 47 Tuc AA, this is the case.
- If it is not unique, then that means you need to tag additional gaps between TOA groups and continue the computation by calling the script again; this will automatically use the acc_WRAPs.dat that now contains the solution(s) found from previous work. You can also do this to check automatically that the solution you obtained, even if unique, stays in fact unique (or within your chi2 threshold) for the remainder of the gaps. Try doing this with 47TucAA.tim.
- This means that, if you choose to do so, you can use dracula.sh as you use sieve.sh: by tagging new gaps progressively (you choose how many at a time), and retaining the file with the solutions for those gaps (acc_WRAPs.dat) as a starting point for the subsequent calculations.
- This flexible sieve.sh-like usage requires more mannual intervention, but it gives you a better idea of when the solution becomes unique, and will also give you an idea of whether your parameter set might become inadequate. This happens when the chi2 of all solutions, even the best ones, starts going up and up. Indeed, the longer the timing baseline, the more parameters you might need (like, for instance, the proper motion, which becomes necessary for multi-year data sets). 
- This progressive usage also has the benefit that you can probe which gaps give you fewer new solutions; generally these are the shorter ones in time, or the ones that are adjacent to a group that is already partially connected. If a new gap gives you way too many solutions, don't worry: just stop the script, put its tag in some other gap, and re-start.


### Usage

* Download dracula.sh, 47TucAA.tim and 47TucAA.par into a directory. If you're starting from scratch, make a file called acc_WRAPs.dat containing three zeros in sequence, with spaces between them. Then edit dracula.sh, as instructed within the script itself. Make it execulable, using

> chmod u+x dracula.sh

and then run it:

> ./dracula.sh

### Known issues

* chi2 can start to blow up to the point where tempo.lis just writes it as a bunch of asterisks, and this confuses the parsing of tempo.lis into sticking your directory listing into WRAPs.dat.

  This can be edited easily in your tempo source code. In a file called newval.f, there is a sequence of lines that read:
  
  
  1108	  format(' Chisqr/nfree: ',f9.2,'/',i5,' = ',f15.9,

  1109	  format(' Chisqr/nfree: ',f10.2,'/',i5,' = ',f15.9,

  1110	  format(' Chisqr/nfree: ',f11.2,'/',i5,' = ',f15.9,


  Replace the f15.9 with f15.4, and compile tempo. We don't need a very high precision in the chi2, but we need to be able to print very large values.
  

* For sieve. sh there is some manual intervention in this process (editing in the PHASEA, PHASEB,... statements in the TOA list, editing the labels in sieve.sh). 
This issue is avoided by the use of the dracula.sh script, unless one chooses to use it as sieve.sh, by naming more and more gaps.

* The tempo runs waste most of the time repeating many steps, like consulting the Earth rotation tables, solar system ephemerides, etc, until we get to the stage where we have the precise vectors between the telescope at the time of the TOA and the Solar System Barycentre. All of that only needs to be run once. My next step will be to use PINT to do these calculations separately, or find a way of making reliable barycentric TOAs, and work with those.

* (Erik Madsen): Personally, I'd have written it in Python, but to each their own!
(Paulo Freire): why use python when very simple shell commands do so well??
 
### Pulsars that have been connected with sieve.sh and dracula.sh (in refereed literature, more unpublished):

- PSR J0024-7205aa (Freire & Ridolfi 2018, MNRAS, 476, 4794)
- PSR J0732+2314 (Martinez et al. 2019, ApJ, 881, 166)
- PSRs J1906+0454, J1921+1929, J1928+1245, J1930+2441 and J1932+1756. PSR J1921+1929 is a gamma-ray MSP (Parent et al. 2019, ApJ, 886, 148)
- New pulsars presented by Cameron et al. (2020, MNRAS, 493, 1063)
- New FAST pulsars timed at Parkes, presented by Cameron et al. (2020, MNRAS, 495, 3515)
- PSRs J1805+0615, J1824+1014, J1908+2105, J2006+0148 and J2052+1219, all of them gamma-ray MSPs (Deneva et al. 2021, ApJ, 909, 6)
- PSR J1536-4948, a gamma-ray MSP (Bhattacharyya et al. 2021, accepted for publication in ApJ, arXiv: 2102:04026v1)
