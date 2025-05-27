Authors: Saskia Hekker, Susmita Das, Zhao Guo, Arthur Le Saux and Noi Shitrit for MESA School Leuven 2025

# Minilab 1: The red-giant bump in low-mass stars

## Section 1: Overview
### Science Goal

As low-mass stars evolve into red-giant stars, there comes a brief phase during their post-main-sequence evolution where two interesting phenomena are observed. First, there is the simultaneous contraction of the core and the expansion of the envelope, known as the *mirror phenomenon*. Next, there occurs a zig-zag in the evolutionary track called the *Red Giant Branch Bump (RGB bump)* where the trend for increasing luminosity is reversed, see [![Fig. 1](https://academic.oup.com/view-large/figure/198891779/staa176fig1.jpg)](https://academic.oup.com/view-large/figure/198891779/staa176fig1.jpg). In this minilab, you will investigate the underlying microphysics that drives these phenomena  using [MESA](https://docs.mesastar.org/en/latest/) and thereby reproduce the work of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract). 

### What you'll learn

The primary purpose of this minilab is to get you more familiar with some topics in MESA beyond the absolute basics, including:

1. Starting a project from a given test case and changing inlist controls
2. Computing a few additional parameters and adding new history columns using <span style="color:purple">``run_star_extras.f90``</span>
3. Customizing <span style="color:purple">``pgstar``</span>

### Using this Guide

Throughout this guide, monospaced text on a black background, like this: <span style="background-color:black"><span style="color:white">`echo "Hello, World!"`</span></span>  represent commands you can type into your terminal and execute, while purple monospaced text like this: <span style="color:purple">``initial_mass=1.0``</span> represent file names or values that you might type into a file.

If you're new to Fortran, here is a short document with [some examples](https://jschwab.github.io/mesa-2016/fortran.html). Don't let yourself get hung up by the Fortran; quickly ask your classmates and the TAs for help! 

Every task comes with a hint or an answer. However, if you have prior experience with MESA, do attempt to complete the task on your own. The complete solution is available <span style="color:red">here</span>.

## Section 2: Getting Started

<span style="color:green">Task 2.1</span>: Create your working directory for this minilab. It could be something like <span style="color:purple">``~/MESASS2025/Day4``</span>.  
Note: you may also choose to place the working directory somewhere other than your home directory. 
<details>
<summary>Hint 2.1</summary>
Much of this should be familiar already; here's how you create your working directory and then change into that directory:
<pre>
mkdir -p ~/MESASS2025/Day4
cd ~/MESASS2025/Day4
</pre>
</details>

<span style="color:green">Task 2.2</span>: We have prepared and provided the test case for you. [Download](https://drive.google.com/file/d/1bCZQii9ZmktXr0k_TAip0tM8w0kqNkFE/view?usp=sharing) it into the <span style="color:purple">``~/MESASS2025/Day4``</span> directory, unpack, and enter this work directory. 

<span style="color:green">Answer 2.2</span>
```fortran
unzip Minilab1.zip
cd Minilab1
```
You are now ready to start the run!

<span style="color:green">Task 2.3</span>: Compile and run the provided work directory.

This directory evolves a solar mass star from the start of the RGB bump upto the end of the RGB bump. Confirm that you can compile and run it. Two default PGPLOT windows (Hertzsprung-Russell Diagram and temparature-density profille) should appear. 

<span style="color:green">Answer 2.3</span>
```fortran
./clean
./mk
./rn
```

This was a test run to ensure everything works fine for you; you do not need to complete the run at this point. When the two windows with plots appear, you may terminate the run using <span style="background-color:black"><span style="color:white">`Ctrl + C`</span></span>.

### Using inlists

 MESA/star currently has five inlist sections. Each section contains the options for a different aspect of MESA.

**star_job**
    options for the program that evolves the star
**eos**
    options for the MESA eos module
**kap**
    options for the MESA kap module
**controls**
    options for the MESA star module
**pgstar**
    options for on-screen plotting 

<span style="color:green">Task 2.4</span>: List the contents of your working directory and identity the number of inlists you see.
<details>
<summary>Hint 2.4</summary>
There are three inlists- inlist, inlist_project and inlist_pgstar. The main inlist points to the inlist_project for the inlist sections: star_job, eos, kap and controls while it points to the inlist_pgstar for plotting options only.
</details>

<span style="color:green">Task 2.5</span>: Open the prepared ``inlist_project`` and answer the following questions: (i) where does the model start its run from? (ii) what is the terminating condition used? (iii) what is the metallicity of the model computed? 
``` fortran 
! inlist to evolve a 1 solar mass star
! For the sake of future readers of this file (yourself included),
! ONLY include the controls you are actually using. DO NOT include
! all of the other controls that simply have their default values.

&star_job
	! begin with a pre-main sequence model
	create_pre_main_sequence_model = .false.

	! begin with saved model
	load_saved_model = .true.
	load_model_filename = 'start_RGBB.mod'

	! save a model at the end of the run
	save_model_when_terminate = .true.
	save_model_filename = 'end_RGBB.mod'
	
	! display on-screen plots
	pgstar_flag = .true.
	
	!pause_before_terminate=.true.
	
/ !end of star_job namelist

&eos
  ! eos options
  ! see eos/defaults/eos.defaults

/ ! end of eos namelist


&kap
  ! kap options
  ! see kap/defaults/kap.defaults
  use_Type2_opacities = .true.
  Zbase = 0.02

/ ! end of kap namelist

&controls
	! starting specifications
	initial_mass = 1.0 ! in Msun units
	initial_z = 0.02
	
	energy_eqn_option = 'eps_grav'
	
	log_L_upper_limit = 1.54
	
	!control output
	history_interval = 1
/ ! end of controls namelist
```

<details>
<summary>Hint 2.5</summary>
(i) The inlist_project loads a saved model called ``start_RGBB.mod`` that was pre-computed to save on computation time and start its run from that particular stage of evolution. (ii) The run stops when the terminating condition of the upper limit of logL reaches 1.54. (iii) The metallicity is Z=0.02. Note that initial_z and Zbase must be kept the same in almost all cases.
</details>

## Section 3: Using Run Star Extras

To activate <span style="color:purple">``run_star_extras.f90``</span>, navigate to the <span style="color:purple">``Minilab1/src``</span> directory and open <span style="color:purple">``run_star_extras.f90``</span> in your text editor of choice. The stock version of <span style="color:purple">``run_star_extras.f90``</span> is quite boring. It "includes" another file which holds the default set of routines. 
```fortran
include 'standard_run_star_extras.inc'
```
The routines defined in the included file are the ones we will want to customize. Because we want these modifications to apply only to this working copy of MESA, and not to MESA as a whole, we want to replace this include statement with the contents of the included file. 

<span style="color:green">Task 3.1</span>: Delete the aforementioned include line and insert the contents of 
```fortran
$MESA_DIR/include/standard_run_star_extras.inc
``` 
in its entirety into <span style="color:purple">``run_star_extras.f90``</span>.

<details>
<summary>Hint 3.1</summary>
A simple copy and paste works here.
</details>

<span style="color:green">Task 3.2</span>: Check that the code compiles.

<span style="color:green">Answer 3.2</span>
```fortran
cd ..
./mk
``` 
If it doesn't compile, double check that you cleanly inserted the file and removed the include line.

Since <span style="color:purple">``run_star_extras.f90``</span> was already introduced in the Day 2 Morning Session in considerable depth, we will now go straight to modifying it for our science test case.

### Section 3.1: Studying the evolution of the RGB bump feature

One of our primary goals is to study the evolution around the RGB bump of (i) the location of the base of the convection zone, (ii) the peak of the burning, and (iii) the mean molecular weight discontinuity as a function of mass ordinate and radius ordinate and thereby, reproduce [![Fig. 4](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg)](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg) of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

#### The base of the convection zone

<span style="color:green">Task 3.3</span>: Output the mass coordinate at the base of the convection zone in your <span style="color:purple">``history.data``</span>.

<details>
<summary>Hint 3.3.1</summary>
Open the <span style="color:purple">``history_columns.list``</span> in your working directory and search for the phrase "conditions at base of largest convection zone".
</details>

<details>
<summary>Hint 3.3.2</summary>
Identify what is the right parameter (cz_bot_mass) and uncomment (that is, remove the "!" at the front of cz_bot_mass) to include it in the output file.
</details>

<span style="color:green">Task 3.4</span>: Output the radius coordinate at the base of the convection zone in your <span style="color:purple">``history.data``</span>.
<details>
<summary>Hint 3.4</summary>
Identify what is the right parameter (cz_bot_radius) and uncomment to include it in the output file.
</details>

<span style="color:green">Task 3.5</span>: While you're at it, check if there exists default history columns for peak of the burning or the mean molecular weight in <span style="color:purple">``history_columns.list``</span>.

<span style="color:green">Answer 3.5</span>: They don't! Thankfully, we can customise our <span style="color:purple">``run_star_extras.f90``</span> to compute additional parameters and add them as new history columns in the <span style="color:purple">``history.data``</span> file.

#### The peak of the burning

The goal here is to identify the zone (k) in the interior of the stellar structure where the nuclear burning (eps_nuc) is maximum and thereby find the mass and the radius corresponding to that zone.

Before making any changes to the <span style="color:purple">``run_star_extras.f90``</span>, take a quick look to identify where additional history columns may be added. The default looks as follows:
```fortran
integer function how_many_extra_history_columns(id)
         integer, intent(in) :: id
         integer :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         how_many_extra_history_columns = 0
    end function how_many_extra_history_columns
      
      
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
         integer, intent(in) :: id, n
         character (len=maxlen_history_column_name) :: names(n)
         real(dp) :: vals(n)
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
         ! note: do NOT add the extras names to history_columns.list
         ! the history_columns.list is only for the built-in history column options.
         ! it must not include the new column names you are adding here.
         

    end subroutine data_for_extra_history_columns
```
> 💡 **Important Fortran Tips:**
 >1. Declare all new variables BEFORE the ```ierr = 0``` statement in the ```data_for_extra_history_columns``` subroutine.
>2. Add the new parameters to be computed and the new history columns AFTER the ```if (ierr /= 0) return``` statement in the ```data_for_extra_history_columns``` subroutine.
>3. Remember to update the right number in ```how_many_extra_history_columns = 0``` as and when you add additional history columns in the ```how_many_extra_history_columns``` function.

<span style="color:green">Task 3.6</span>: Compute the mass of the zone where the nuclear burning is at its peak and add it as a new column in your <span style="color:purple">``history.data``</span>.

<details>
<summary>Hint 3.6.1</summary>
You can use maxloc to identify the zone (k) with maximum nuclear burning (eps_nuc):  
k=maxloc(s% eps_nuc, dim=1)
</details>

<details>
<summary>Hint 3.6.2</summary>
The mass corresponding to that particular zone is then s% m(k)/Msun; assign it to a new variable mass_max_eps_nuc
</details>

<details>
<summary>Hint 3.6.3</summary>
The new history column can be added as:
names(1) = "mass_max_eps_nuc"
vals(1) = mass_max_eps_nuc
</details>

<details>
<summary>Hint 3.6.4</summary>
Remember to set how_many_extra_history_columns = 1 at this point.
</details>

<details>
<summary>Answer 3.6</summary>
At this point, the partial solution to your run_star_extras.f90 file should look like this:

```fortran
integer function how_many_extra_history_columns(id)
         integer, intent(in) :: id
         integer :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         how_many_extra_history_columns = 1
    end function how_many_extra_history_columns
      
      
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
         integer, intent(in) :: id, n
         character (len=maxlen_history_column_name) :: names(n)
         real(dp) :: vals(n)
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         integer :: k
	 real(dp) ::mass_max_eps_nuc
	
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
         k=maxloc(s% eps_nuc, dim=1)       
    	 mass_max_eps_nuc = s% m(k)/Msun
         
         ! note: do NOT add the extras names to history_columns.list
         ! the history_columns.list is only for the built-in history column options.
         ! it must not include the new column names you are adding here.
         
         names(1) = "mass_max_eps_nuc"
	 vals(1) = mass_max_eps_nuc

    end subroutine data_for_extra_history_columns
```
</details>

<span style="color:green">Task 3.7</span>: Compute the radius of the zone where the nuclear burning is at its peak and add it as a new column in your <span style="color:purple">``history.data``</span>.

<details>
<summary>Hint 3.7</summary>
You already have the zone (k) where nuclear burning is at its peak. Simply include the radius of that zone using s% r(k)/Rsun following the steps as before.
</details>

<details>
<summary>Answer 3.7</summary>
At this point, the partial solution to your run_star_extras.f90 file should look like this:

```fortran
integer function how_many_extra_history_columns(id)
         integer, intent(in) :: id
         integer :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         how_many_extra_history_columns = 2
    end function how_many_extra_history_columns
      
      
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
         integer, intent(in) :: id, n
         character (len=maxlen_history_column_name) :: names(n)
         real(dp) :: vals(n)
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         integer :: k
	 real(dp) ::mass_max_eps_nuc, radius_max_eps_nuc
	
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
         k=maxloc(s% eps_nuc, dim=1)       
    	 mass_max_eps_nuc = s% m(k)/Msun
         radius_max_eps_nuc = s% r(k)/Rsun
         
         ! note: do NOT add the extras names to history_columns.list
         ! the history_columns.list is only for the built-in history column options.
         ! it must not include the new column names you are adding here.
         
         names(1) = 'mass_max_eps_nuc'
	 vals(1) = mass_max_eps_nuc

         names(2) = 'radius_max_eps_nuc'
	 vals(2) = radius_max_eps_nuc

    end subroutine data_for_extra_history_columns
```
</details>

<span style="color:green">Task 3.8</span>: After making changes to the <span style="color:purple">``run_star_extras.f90``</span>, always check that the code compiles.

<span style="color:green">Answer 3.8</span>
```fortran
cd ..
./mk
``` 

#### The mean molecular weight discontinuity

We will now include a new Fortran subroutine ```locdiscontinuity``` in the <span style="color:purple">``run_star_extras.f90``</span> to identify the location of discontinuities in the mean molecular weight profile (with respect to the hydrogen abundance, *X*) of a stellar model. These discontinuities are important indicators of nuclear burning shells, convective boundaries, or mixing events in stellar evolution.

<span style="color:green">Task 3.9</span>: Compute the mass and the radius at the location of the mean molecular weight discontinuity.

<span style="color:green">Answer 3.9</span>: If you'd like to attempt on your own, please do so. However, since this is a little advanced, we have added the answer directly for your ease. Here is the snippet of how your <span style="color:purple">``run_star_extras.f90``</span> should look like:
```fortran
...
!calculate location of mean molecular weight discontinuity
subroutine locdiscontinuity(id,nz,sdisc1,sdisc2,sdiscdif1,sdiscdif2) 
      integer, intent(in) :: id, nz
      integer, intent(out) :: sdisc1, sdisc2
      real(dp), intent(out) :: sdiscdif1, sdiscdif2
      real(dp) :: x(nz), dif(nz-1)
      integer :: ierr, xs, xsb,k
      type (star_info), pointer :: s
      ierr = 0
      call star_ptr(id, s, ierr)
      if (ierr /= 0) return
 
       x = s% X(:nz) 
       dif = x(:size(x)-1)-x(2:)
       k = maxloc(s% eps_nuc(:nz), dim=1)
       xsb = k
       sdisc1=maxloc(dif, dim=1)
       sdiscdif1=maxval(dif)
       sdisc2=maxloc(dif(:xsb-1), dim=1)
       sdiscdif2=maxval(dif(:xsb-1))
    end subroutine locdiscontinuity

integer function how_many_extra_history_columns(id)
         integer, intent(in) :: id
         integer :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         how_many_extra_history_columns = 8
    end function how_many_extra_history_columns
      
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
         integer, intent(in) :: id, n
         character (len=maxlen_history_column_name) :: names(n)
         real(dp) :: vals(n)
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         integer :: k, nz, sdisc1,sdisc2
	 real(dp) ::mass_max_eps_nuc, radius_max_eps_nuc, disc1, disc2
	
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
          k=maxloc(s% eps_nuc, dim=1)       
	 mass_max_eps_nuc = s% m(k)/Msun
	 radius_max_eps_nuc = s% r(k)/Rsun	
         
         ! note: do NOT add the extras names to history_columns.list
         ! the history_columns.list is only for the built-in history column options.
         ! it must not include the new column names you are adding here.
         
          names(1) = 'mass_max_eps_nuc'
	 vals(1) = mass_max_eps_nuc
	
	 names(2) = 'radius_max_eps_nuc'
	 vals(2) = radius_max_eps_nuc
	
	 nz = s% nz
         
         names(3) = 'mdisc1'
         names(4) = 'rdisc1'
         names(5) = 'mdisc2'
         names(6) = 'rdisc2'
         names(7) = 'disc1'
         names(8) = 'disc2'
     
         call locdiscontinuity(id,nz,sdisc1,sdisc2,disc1,disc2)
         vals(3) = s% m(sdisc1)/(Msun)
         vals(4) = s% r(sdisc1)/(Rsun)
         vals(5) = s% m(sdisc2)/(Msun)
         vals(6) = s% r(sdisc2)/(Rsun)
         vals(7) = disc1
         vals(8) = disc2
    end subroutine data_for_extra_history_columns
...
``` 

<details>
<summary>Explanation 3.9</summary>
An explanation of the Fortran subroutine ```locdiscontinuity``` is included here:  
<pre>

!=======================================================================
! Subroutine: locdiscontinuity
! Purpose: Identify locations and magnitudes of composition discontinuities
!          in a stellar model, often associated with hydrogen abundance (X).
!=======================================================================
subroutine locdiscontinuity(id, nz, sdisc1, sdisc2, sdiscdif1, sdiscdif2)

  ! Input arguments
  integer, intent(in) :: id         ! Model identifier
  integer, intent(in) :: nz         ! Number of zones

  ! Output arguments
  integer, intent(out) :: sdisc1    ! Index of strongest overall discontinuity
  integer, intent(out) :: sdisc2    ! Index of strongest outer-layer discontinuity
  real(dp), intent(out) :: sdiscdif1 ! Magnitude of strongest overall discontinuity
  real(dp), intent(out) :: sdiscdif2 ! Magnitude of strongest outer-layer discontinuity

  ! Local variables
  real(dp) :: x(nz)                 ! Composition profile (e.g. hydrogen abundance)
  real(dp) :: dif(nz-1)             ! Backward differences between mesh points
  real(dp) :: max_eps_h_m
  integer :: ierr, xs, xsb, k
  type (star_info), pointer :: s   ! Pointer to stellar structure data

  ierr = 0
  call star_ptr(id, s, ierr)        ! Get pointer to star data
  if (ierr /= 0) return             ! Exit if pointer setup fails

  x = s% X(:nz)                     ! Load hydrogen abundance profile
  dif = x(:size(x)-1) - x(2:)       ! Compute differences between adjacent zones

  k = maxloc(s% eps_nuc(:nz), dim=1) ! Find zone of peak nuclear energy generation
  xsb = k                            ! (used as proxy for peak H burning)

  sdisc1 = maxloc(dif, dim=1)       ! Global maximum discontinuity location
  sdiscdif1 = maxval(dif)           ! Its magnitude

  sdisc2 = maxloc(dif(:xsb-1), dim=1) ! Max discontinuity above burning zone
  sdiscdif2 = maxval(dif(:xsb-1))     ! Its magnitude

end subroutine locdiscontinuity

!=======================================================================
! Extracting physical information from discontinuities
!=======================================================================
call locdiscontinuity(id, nz, sdisc1, sdisc2, disc1, disc2)

vals(3) = s% m(sdisc1) / (Msun)
! Mass (in solar units) of the strongest discontinuity

vals(4) = s% r(sdisc1) / (Rsun)
! Radius (in solar units) of the strongest discontinuity

vals(5) = s% m(sdisc2) / (Msun)
! Mass (in solar units) of the outer-layer discontinuity

vals(6) = s% r(sdisc2) / (Rsun)
! Radius (in solar units) of the outer-layer discontinuity

vals(7) = disc1
! Magnitude of the strongest overall discontinuity

vals(8) = disc2
! Magnitude of the strongest discontinuity above the burning core
</pre>
</details>

<span style="color:green">Task 3.10</span>: Once again, after making changes to the <span style="color:purple">``run_star_extras.f90``</span>, check that the code compiles.

<span style="color:green">Answer 3.10</span>
```fortran
cd ..
./mk
``` 

Great work! You have now included most of the parameters that are required to reproduce [![Fig. 4](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg)](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg) of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

### Section 3.2: The variation of the 'gravothermal' energy generation rate with age

We also want to study the variation of the 'gravothermal' energy generation rate  \( \epsilon_g \) at the base of the convection zone as a function of age around the evolution of the RGB bump and thereby reproduce [![Fig. 6](https://academic.oup.com/view-large/figure/198891806/staa176fig6.jpg)](https://academic.oup.com/view-large/figure/198891806/staa176fig6.jpg) of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

In Section 3.1, we had already seen how to include the mass and the radius ordinates at the base of the convection zone. We only had to uncomment the history column names in the <span style="color:purple">``history_columns.list``</span> since they are computed as default history columns. However, this holds clue to how one many compute new additional parameters at the base of the conevction zone. To look at how the mass and the radius ordinates were computed at the base of the convection zone, open the file:
```fortran
$MESA_DIR/star/private/history.f90
``` 
and search for <span style="color:purple">``cz_bot_mass``</span> or <span style="color:purple">``cz_bot_radius``</span>.

<span style="color:green">Task 3.11</span>: Compute \( \epsilon_g \) at the base of the convection zone (cz_eps_grav) and add it as a new column in your <span style="color:purple">``history.data``</span>.

<details>
<summary>Hint 3.11.1</summary>
The eps_grav parameter can be accessed as s% eps_grav_ad(k)% val in <span style="color:purple">``run_star_extras.f90``</span>.
</details>

<details>
<summary>Hint 3.11.2</summary>
Here is the snippet of code that can be used in the <span style="color:purple">``run_star_extras.f90``</span>:
<pre>
if (s% largest_conv_mixing_region /= 0) then
        k = s% mixing_region_bottom(s% largest_conv_mixing_region)
        cz_eps_grav = s% eps_grav_ad(k)% val
    end if
</pre>
</details>

<span style="color:green">Answer 3.11</span>: Here is the snippet of how your <span style="color:purple">``run_star_extras.f90``</span> should look like:
``` fortran
...
integer function how_many_extra_history_columns(id)
         integer, intent(in) :: id
         integer :: ierr
         type (star_info), pointer :: s
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         how_many_extra_history_columns = 9
    end function how_many_extra_history_columns
      
subroutine data_for_extra_history_columns(id, n, names, vals, ierr)
         integer, intent(in) :: id, n
         character (len=maxlen_history_column_name) :: names(n)
         real(dp) :: vals(n)
         integer, intent(out) :: ierr
         type (star_info), pointer :: s
         integer :: k, nz, sdisc1,sdisc2
	real(dp) ::mass_max_eps_nuc, radius_max_eps_nuc, disc1, disc2, cz_eps_grav
	
         ierr = 0
         call star_ptr(id, s, ierr)
         if (ierr /= 0) return
         
          k=maxloc(s% eps_nuc, dim=1)       
	mass_max_eps_nuc = s% m(k)/Msun
	radius_max_eps_nuc = s% r(k)/Rsun	

        if (s% largest_conv_mixing_region /= 0) then
                k = s% mixing_region_bottom(s% largest_conv_mixing_region)
                cz_eps_grav = s% eps_grav_ad(k)% val
             end if
         
         ! note: do NOT add the extras names to history_columns.list
         ! the history_columns.list is only for the built-in history column options.
         ! it must not include the new column names you are adding here.
         
          names(1) = 'mass_max_eps_nuc'
	vals(1) = mass_max_eps_nuc
	
	names(2) = 'radius_max_eps_nuc'
	vals(2) = radius_max_eps_nuc
	
	nz = s% nz
         
         names(3) = 'mdisc1'
         names(4) = 'rdisc1'
         names(5) = 'mdisc2'
         names(6) = 'rdisc2'
         names(7) = 'disc1'
         names(8) = 'disc2'
     
         call locdiscontinuity(id,nz,sdisc1,sdisc2,disc1,disc2)
         vals(3) = s% m(sdisc1)/(Msun)
         vals(4) = s% r(sdisc1)/(Rsun)
         vals(5) = s% m(sdisc2)/(Msun)
         vals(6) = s% r(sdisc2)/(Rsun)
         vals(7) = disc1
         vals(8) = disc2

         names(9) = 'cz_eps_grav'
	vals(9) = cz_eps_grav
    end subroutine data_for_extra_history_columns
...
```
<span style="color:green">Task 3.12</span>: As always, after making changes to the <span style="color:purple">``run_star_extras.f90``</span>, check that the code compiles.
<span style="color:green">Answer 3.12</span>
```fortran
cd ..
./mk
``` 

This is what your final <span style="color:purple">``run_star_extras.f90``</span>  should look like.

Now that you have all the parameters, you are essentially ready to start the run! If you're short on time, you may grab the final <span style="color:purple">``inlist_pgstar``</span> <span style="color:red">``here``</span> and jump straight to Section 5. However, if you're interested and have time, let's customise the <span style="color:purple">``inlist_pgstar``</span> in the next section for a better understanding of how the stellar structure/interiors change as the star evolves around the RGB bump.

## Section 4: Customizing pgstar 

 <span style="color:purple">``pgstar``</span> is a built-in feature of MESA that allows for real-time graphical insight into how your model is evolving. While the plots it generates are usually not suitable for publication, being able to "see" your model evolve can be an invaluable tool in developing intuition and diagnosing issues. Additionally, you can easily string frames of <span style="color:purple">``pgstar``</span> output into a movie after a simulation, which is great for presentations and group meetings!

 <span style="color:purple">``pgstar``</span> is comprised of several building blocks that we can sort into several [slightly-overlapping] categories.

**History Plots**
    Plots that show one or two quantities from the history output plotted against a monotonically-increasing quantity, like `star_age` or `model_number`. These quantities must all be saved in <span style="color:purple">``history.data``</span>, and the resolution will depend on how often data is written to  <span style="color:purple">``history.data``</span>. 
**Profile Plots**
    Plots that show one or two quantities that could be output in profiles against another profile quantity, often pressure, mass coordinate, or radius. Note that these do not need to be in <span style="color:purple">``profile.columns``</span> since they do not need to persist over multiple steps. 
**Special Plots**
    A collection of "one-off" plots with special capabilities. Examples include Kippenhahn diagrams, echelle diagrams, a nuclear network diagram, and a temperature-density profile. These can often be customized to a degree, but they are not as flexible as profile and history plots. 
**Text Summaries**
    Grids of name/value pairs that show scalar values associated with the current timestep. Examples include model number luminosity, and helium core mass. No graphics here; just text. 

Most often, you'll deal with a grid or dashboard that contains many individual single- or multi-panel plots and/or text summaries arranged into a single window. We'll explore this next. 

**Goal:** In this section, in addition to the default Hertzsprung-Russell Diagram and temperature-density profile, we will also visualise the variation of the specific entropy, mean molecular weight, density, pressure, temperature and \( \epsilon_g \) in the stellar interiors as a function of mass fraction.

<span style="color:green">Task 4.1</span>: Open the <span style="color:purple">``inlist_pgstar``</span> in your favourite editor and turn the `HR_win_flag` and `TRho_Profile_win_flag` to `false` to prevent their individual PGPLOT windows.
<span style="color:green">Answer 4.1</span>
```fortran
 HR_win_flag = .false.
 TRho_Profile_win_flag = .false.
``` 

<span style="color:green">Task 4.2</span>: Include the first profile panel in the <span style="color:purple">``inlist_pgstar``</span> to display specific entropy, mean molecular weight, density as a function of mass fraction.
<span style="color:green">Answer 4.2</span>
```fortran
 ! Profile Panels 1

  Profile_Panels1_title = 'Profile Panels'
  Profile_Panels1_xaxis_name = 'mass'
  Profile_Panels1_xmin = 0
  Profile_Panels1_xmax = 1

  Profile_Panels1_show_mix_regions_on_xaxis = .false.
  Profile_Panels1_yaxis_name(1) = 'entropy'
  Profile_Panels1_yaxis_name(2) = 'mu'
  Profile_Panels1_yaxis_name(3) = 'logRho'
  
  Profile_Panels1_other_yaxis_name(1) = ''
  Profile_Panels1_other_yaxis_name(2) = ''

  Profile_Panels1_num_panels = 3
``` 

<span style="color:green">Task 4.3</span>: Include the second profile panel in the <span style="color:purple">``inlist_pgstar``</span> to display \( \epsilon_g \), temperature and pressure as a function of mass fraction.
<details>
<summary>Hint 4.3</summary>
The respective profile column names are `eps_grav`, `logT` and `logP`.
</details>

<span style="color:green">Task 4.4</span>: Include a few important parameters as part of the text block in the <span style="color:purple">``pgstar``</span>. 

<span style="color:green">Answer 4.4</span>
```fortran
 ! Text Summary 1

  Text_Summary1_name(1,1) = 'model_number'
  Text_Summary1_name(2,1) = 'log_star_age'
  Text_Summary1_name(3,1) = 'log_dt'
  Text_Summary1_name(4,1) = 'log_L'
  Text_Summary1_name(5,1) = 'log_Teff'
  Text_Summary1_name(6,1) = 'log_R'
  Text_Summary1_name(7,1) = 'log_g'
  Text_Summary1_name(8,1) = 'log_surf_cell_P'

  Text_Summary1_name(1,2) = 'star_mass'
  Text_Summary1_name(2,2) = 'log_abs_mdot'
  Text_Summary1_name(3,2) = 'he_core_mass'
  Text_Summary1_name(4,2) = 'co_core_mass'
  Text_Summary1_name(5,2) = 'h_rich_layer_mass'
  Text_Summary1_name(6,2) = 'he_rich_layer_mass'
  Text_Summary1_name(7,2) = ''
  Text_Summary1_name(8,2) = ''

  Text_Summary1_name(1,3) = 'log_cntr_T'
  Text_Summary1_name(2,3) = 'log_cntr_Rho'
  Text_Summary1_name(3,3) = 'log_center_P'
  Text_Summary1_name(4,3) = 'center h1'
  Text_Summary1_name(5,3) = 'center he4'
  Text_Summary1_name(6,3) = 'center c12'
  Text_Summary1_name(7,3) = 'center n14'
  Text_Summary1_name(8,3) = 'center o16'

  Text_Summary1_name(1,4) = 'log_Lnuc'
  Text_Summary1_name(2,4) = 'log_Lneu'
  Text_Summary1_name(3,4) = 'log_LH'
  Text_Summary1_name(4,4) = 'log_LHe'
  Text_Summary1_name(5,4) = 'log_LZ'
  Text_Summary1_name(6,4) = 'num_zones'
  Text_Summary1_name(7,4) = 'num_retries'
  Text_Summary1_name(8,4) = ''
``` 

<span style="color:green">Task 4.4</span>: Now combine all the grid information to activate the customised <span style="color:purple">``pgstar``</span> dashboard. The parameters are mostly self-explanatory and are adopted from <span style="color:purple">``$MESA_DIR/star/defaults/pgstar.defaults``</span>.

<span style="color:green">Answer 4.4</span>
```fortran
 ! Grid1,information to combine all plots into Grid1

  Grid1_win_flag = .true.
  Grid1_win_width = 12 
  Grid1_win_aspect_ratio = 0.55
  Grid1_xleft = 0.06
  Grid1_xright = 0.95
  Grid1_ybot = 0.08
  Grid1_ytop = 0.92
  Grid1_title = ''

  Grid1_num_cols = 3
  Grid1_num_rows = 8
  Grid1_num_plots = 5

  Grid1_plot_name(1) = 'TRho_Profile'
  Grid1_plot_row(1) = 1
  Grid1_plot_rowspan(1) = 3
  Grid1_plot_col(1) = 1
  Grid1_plot_colspan(1) = 1
  Grid1_plot_pad_left(1) = 0
  Grid1_plot_pad_right(1) = 0.05
  Grid1_plot_pad_top(1) = 0
  Grid1_plot_pad_bot(1) = 0.1
  Grid1_txt_scale_factor(1) = 0.7

  Grid1_plot_name(2) = 'HR'
  Grid1_plot_row(2) = 4
  Grid1_plot_rowspan(2) = 3
  Grid1_plot_col(2) = 1
  Grid1_plot_colspan(2) = 1
  Grid1_plot_pad_left(2) = 0
  Grid1_plot_pad_right(2) = 0.05
  Grid1_plot_pad_top(2) = 0.03
  Grid1_plot_pad_bot(2) = 0
  Grid1_txt_scale_factor(2) = 0.7
    
  Grid1_plot_name(3) = 'Profile_Panels1'
  Grid1_plot_row(3) = 1
  Grid1_plot_rowspan(3) = 6
  Grid1_plot_col(3) = 2
  Grid1_plot_colspan(3) = 1
  Grid1_plot_pad_left(3) = 0.04
  Grid1_plot_pad_right(3) = 0.03
  Grid1_plot_pad_top(3) = 0
  Grid1_plot_pad_bot(3) = 0
  Grid1_txt_scale_factor(3) = 0.65
  
  Grid1_plot_name(4) = 'Text_Summary1'
  Grid1_plot_row(4) = 7
  Grid1_plot_col(4) = 1
  Grid1_plot_rowspan(4) = 2
  Grid1_plot_colspan(4) = 3
  Grid1_plot_pad_left(4) = -0.08
  Grid1_plot_pad_right(4) = 0
  Grid1_plot_pad_top(4) = 0.08
  Grid1_plot_pad_bot(4) = -0.05
  Grid1_txt_scale_factor(4) = 0.25
  
  Grid1_plot_name(5) = 'Profile_Panels2'
  Grid1_plot_row(5) = 1
  Grid1_plot_rowspan(5) = 6
  Grid1_plot_col(5) = 3
  Grid1_plot_colspan(5) = 1
  Grid1_plot_pad_left(5) = 0.06
  Grid1_plot_pad_right(5) = 0.02
  Grid1_plot_pad_top(5) = 0.0
  Grid1_plot_pad_bot(5) = 0
  Grid1_txt_scale_factor(5) = 0.65

  Grid1_file_flag = .true.
  Grid1_file_dir = 'png'
  Grid1_file_prefix = 'rgbb_'
  Grid1_file_interval = 1
  Grid1_file_width = -1
  Grid1_file_aspect_ratio = -1
``` 

Your final `inlist_pgstar` should look like this: 

## Section 5: Putting it all together

You're now finally ready to start the run!

<span style="color:green">Task 5.1</span>: Start the run
<span style="color:green">Answer 5.1</span>
```fortran
./mk
./rn
``` 

The customised PGPLOT window should look something like this:
add pgplot screenshot here

After the run terminates, you're ready to plot and reproduce the figures of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

<span style="color:green">Task 5.2</span>: Use this Google Colab [notebook](https://colab.research.google.com/drive/1bc6Wkne8K6Abciy7aYEnBawy9eir7XZW?usp=sharing) to upload your <span style="color:purple">``history.data``</span> and plot (i) the evolution around the RGB bump of the location of the base of the convection zone, the peak of the burning, and the mean molecular weight discontinuity as a function of mass ordinate and radius ordinate and compare your output plot with [![Fig. 4](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg)](https://academic.oup.com/view-large/figure/198891802/staa176fig4.jpg) of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

<span style="color:green">Answer 5.2</span>
Include figure here.

<span style="color:green">Task 5.3</span>: Using the same Google Colab notebook, plot the variation of \( \epsilon_g \) at the base of the convection zone as a function of age and compare your output plot with [![Fig. 6](https://academic.oup.com/view-large/figure/198891806/staa176fig6.jpg)](https://academic.oup.com/view-large/figure/198891806/staa176fig6.jpg) of [Hekker et al. (2020)](https://ui.adsabs.harvard.edu/abs/2020MNRAS.492.5940H/abstract).

<span style="color:green">Answer 5.3</span>
Include figure here.
