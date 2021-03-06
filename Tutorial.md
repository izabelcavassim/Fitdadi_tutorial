---
title: "Fitdadi tutorial"
author: "Maria Izabel Cavassim Alves"
date: "7/19/2021"
output:
  html_document: default
  pdf_document: default
---

* [dfe inference](https://dadi.readthedocs.io/en/latest/user-guide/dfe-inference/)
* [demographic inference](https://dadi.readthedocs.io/en/latest/examples/YRI_CEU/YRI_CEU/)
* [dadi website](https://dadi.readthedocs.io/en/latest/)
* [conda website](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html)
* [maximum likelihood analyses](https://en.wikipedia.org/wiki/Maximum_likelihood_estimation)


```{r setup, include=FALSE}
knitr::opts_chunk$set(echo = TRUE)
```

## Accessing the cluster

If you have a mac you can access the cluster through ssh

```{bash, eval=F}
ssh your_name@hoffman2.idre.ucla.edu
```

Otherwise you access through XXXXX

## Installing miniconda

Miniconda is an environment manager tool that makes it easier for us to install software including dadi and many other bioinformatic tools. So I recommend that first of all you guys install miniconda as the following in your directories, by typing the following in the terminal:

```{bash, eval=F}
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh
```

Now try to run python as:

```{bash, eval=F}
python
```
if it worked, you have successfully managed to install conda :D ! 

Now that we have conda installed, we will create an environment especially for this given project. You can call it anything, and I will call it "summer_project". In this project we will add every software needed for our analyses during the summer project, so whenever you work on the cluster you will have those packages organized and ready to be used in the given environment. And the most import, you will have version control of your analyses!

To do so, we need to call conda and tell it to create a new environment as:

```{bash, eval=F}
conda create --name summer_project
```

To learn more about what conda can do, please access [here](https://conda.io/projects/conda/en/latest/user-guide/tasks/manage-environments.html).

Now that we have created our conda environment to be able to use it we need to **"activate"** it. To do so we type:

```{bash, eval=F}
conda activate summer_project
```

if you have successfully installed and activated this environment you will see at the left most side of the screen the name of the project followed by your username. 

Perfect! Now we can install **inside of this environment** all the other packages we will be using to run fitdadi. 

We start it by installing dadi. If we look at the website for the dadi software [here](https://dadi.readthedocs.io/en/latest/user-guide/installation/), they recommend that we use the following code:

```{bash, eval=F}
conda install -c conda-forge dadi
```

It requires that we install multiple packages (dependencies), you just need to type 'y' and enter to proceed with the installation. 
To check whether dadi was successfully installed, we can just call the software on terminal. Since it is a python module, than you should first call python:

```{bash, eval=F}
python
import dadi
```

If it does not complain, then you have succeed on its instalation. 

As you may know dadi is going to be useful in our analyses for estimating specific demographic parameters. The idea here is that it???s important to separate real demographic events from selection events. It is important to have in mind that demographic events will affect the genome differently than selection, while bottlenecks (population contraction) and population expansions are expected to affect neutral diversity and coding sequences **equally**, purifying selection should mainly affect the coding regions. Hence, we are going to conduct demographic inference from neutral regions far from genes (synonymous SFS) to correct for demography+linked selection, and the non-synonymous SFS to look for purifying selection.

The demographic part we conduct with dadi, the selection part we condudct with fitdadi. They are both python modules, that we will need to import when writing our python scripts. 

So let's install fitdadi. The easiest way to do it so is by downloading the 'Selection.py' script from its github repository. But lets first create a folder that will contain all our scripts: as:

```{bash, eval=F}
mkdir scripts_summer_course
cd scripts_summer_course
wget https://raw.githubusercontent.com/LohmuellerLab/fitdadi/master/Selection.py
```

To check if both modules are working then you can just open python and import both dadi and fitdadi's Selection module by:

```{bash, eval=F}
python
import dadi
import Selection
```

You will see that we will get some errors because the Selection.py script has its annotation in python2 and our environment is installed with python3. To fix this, we will edit the Selection.py script manually, and for every 'print statement' we will include parenthesis as:

```{bash, eval=F}
from:
print '{0}: {1}'.format(ii, gamma)

to:

print('{0}: {1}').format(ii, gamma)
```

to edit I will be using emacs:

```{bash, eval=F}
emacs Selection.py
```

Edit lines 80 and 96 as shown above. 
Try to import the module again through python. Type ctrl X ctrl S to have your modifications and then type ctrl z to leave the file. 

To make life easier for you guys, I have also included the Selection.py script [here](https://github.com/izabelcavassim/Fitdadi_tutorial/blob/master/Selection.py). Note that to download the raw script you can press the *Raw* botton around the up right corner of the screen. Once you save it in your computer you should transfer it to the cluster. 

For mac users it is convenient to use the software [Cyberduck](https://cyberduck.io/) for transferring between the cluster and your own machine.  

## Getting familiar with Fitdadi

As I said before, dadi will be used for inference of demographic parameters and fitdadi for computation of selection parameters. Let's first focus on estimating demographic parameters.

We will be using the synonymous site frequency spectrum (SFS) computed by Kim et al., 2017 from the 1000 genomes. It is based on the present european/asian population, and we know that these populations have gone through a two-epoch model: bottleneck -> recovery -> exponential growth.

Let's start creating our script by:
```{bash, eval = F}
touch dadi_fitdadi.py

# open the file
emacs dadi_fitdadi.py
```

now copy and paste the following:

```{python, eval=F, python.reticulate=F}
import dadi
import Selection
import pickle
import numpy

def eur_demog(params, ns, pts):
	"""
	generic european/asian demographic model
	bottleneck -> recovery -> exponential growth
	"""
	N1, T1, N2, T2, NC, TC = params
	dadi.Integration.timescale_factor = 0.001
	xx = dadi.Numerics.default_grid(pts)
	phi = dadi.PhiManip.phi_1D(xx)
	phi = dadi.Integration.one_pop(phi, xx, T1, N1)
	phi = dadi.Integration.one_pop(phi, xx, T2, N2)
	dadi.Integration.timescale_factor = 0.000001
	nu_func = lambda t: N2*numpy.exp(numpy.log(NC/N2)*t/TC)
	phi = dadi.Integration.one_pop(phi, xx, TC, nu_func)
	fs = dadi.Spectrum.from_phi(phi, ns, (xx,))
	return fs
```

now, I want you guys to go thgrough the explanation of the website [here](https://dadi.readthedocs.io/en/latest/examples/YRI_CEU/YRI_CEU/), and I want you to look at that script and make some comments on your own script.

What do you find?

```{python, eval=F, python.reticulate=F}
import dadi
import Selection
import pickle
import numpy

def eur_demog(params, ns, pts):
	"""
	generic european/asian demographic model
	bottleneck -> recovery -> exponential growth
	"""
	N1, T1, N2, T2, NC, TC = params # These are the population sizes and times between events. So N1 is ancestral population size, 
	# T1 is the scaled time (in units of 2*Na generations) between ancestral population growth and the recovery.
	# TC is the time between the recovery and present
	dadi.Integration.timescale_factor = 0.001 # scaling factor to speed the computation
	xx = dadi.Numerics.default_grid(pts) # number of points to be used during the integration
	
	phi = dadi.PhiManip.phi_1D(xx) # phi for the equilibrium ancestral population size, phi is the density of mutations within the populations at given frequencies. 
	phi = dadi.Integration.one_pop(phi, xx, T1, N1) 
	phi = dadi.Integration.one_pop(phi, xx, T2, N2) # phi for the recovery event (population growth)
	
	dadi.Integration.timescale_factor = 0.000001 # scaling factor to speed the computation
	
	nu_func = lambda t: N2*numpy.exp(numpy.log(NC/N2)*t/TC) # the exponential growth function
	phi = dadi.Integration.one_pop(phi, xx, TC, nu_func)  # phi for the exponential growth
	
	# Finally, calculate the spectrum.
	fs = dadi.Spectrum.from_phi(phi, ns, (xx,))
	return fs
```

More about the demographic model specification can be found [here](https://dadi.readthedocs.io/en/latest/user-guide/specifying-a-model/).

Ok, now that we have a function that exemplifies the specific demographic model we wanna test our data against, we can now input the synonymous site frequency spectrum (as our data) and use it to estimate our demographic parameters. So to say we are trying to maximize a likelihood function given the data.

Keep adding into your dadi_fitdadi.py script
```{python, eval=F, python.reticulate=F}
#load 1000 genomes synonymous SFS
infilename = "1kG_syn.csv"
syn_sfs = numpy.genfromtxt(infilename,delimiter=",")
syn_sfs = dadi.Spectrum(syn_sfs)
syn_sfs = syn_sfs.fold()

#set initial parameter guesses, convert to coalescent units
Nanc = 10085.
N1 = 2111.693671/(2*Nanc)
N2 = 22062.08869/(2*Nanc)
NC = 652465.1415/(2*Nanc)
T1 = 0.01 #T1 bottleneck time is fixed !!!!!!!!!!!!!!!!!!!!!!!!!!!!!!
T2 = (1553.2561-235.47597)/(2*Nanc)
TC = 235.47597/(2*Nanc)

#setup dadi optimization stuff
pts_l = [1000, 1400, 1800] # the number of grid points used in the calculation.
func = eur_demog # our demographic model function
func_ex = dadi.Numerics.make_extrap_log_func(func) #Generate a version of func that extrapolates to infinitely many gridpoints.
params = [N1, T1, N2, T2, NC, TC] # our parameters
lower_bound = [1e-2, 0.01, 1e-2, 1e-3, 1, 1e-3] # boundaries
upper_bound = [10, 0.01, 10, 0.5, 200, 0.5]
fixed_params = [None, 0.01, None, None, None, None]

# fit demographic model
# need to make sure parameters are a good fit
# i usually run 25-30 runs and make sure they've converged

# randomize starting point
p0 = dadi.Misc.perturb_params(params, upper_bound=upper_bound)
p0[2] = 0.01

print("This is a initial random point")
print(p0)

```

It is never a good idea to just run stuff on the terminal on the cluster (since that can slow down the cluster). Instead what we can do is to request a node of the cluster for ourselves to play around. To do so you need to type the following:

```{bash, eval = F}
qrsh -l h_data=8G,h_rt=4:00:00
```
This will give us access to a node to play with for 4 hours, once you are there you can now run your python script!

What values do you get? Note that we are just choosing at random a starting point for the optimization. In order for us to actually optimize we need to give boundaries and a initial point (our p0). 

continue adding the following chunk of code to your script:

```{python, eval=F, python.reticulate=F}
# optimize
popt = dadi.Inference.optimize_log(p0, syn_sfs, func_ex, pts_l, 
								   lower_bound=lower_bound, 
								   upper_bound=upper_bound,
								   fixed_params=fixed_params,
								   verbose=len(p0), maxiter=30)


#best fit should be similar to
#popt=[8.45886500e-02,1.00000000e-02,1.09948173e+00,7.04035448e-02,5.32986095e+01,2.00795456e-02]

print("This is the demographic model")
print(popt)

```
To learn more about the optmization procedure please read [here](https://dadi.readthedocs.io/en/latest/user-guide/simulation-and-fitting/)

Once we have found the optimum demographic parameters, we now should be able to compute the population scale diversity (theta_s). Because this method only fits the proportional SFS, theta_S is estimated with the dadi.Inference.optimal_sfs_scaling method. Then, we multiply theta_S by 2.31 to get theta_NS, i.e. theta_S???2.31=theta_NS.        

## Computing synonymous theta   
```{python, eval=F, python.reticulate=F}
theta_s = dadi.Inference.optimal_sfs_scaling(func_ex(popt, syn_sfs.sample_sizes, pts_l),syn_sfs)

print("This is theta synonymous")
print(theta_s)
```

## Computing the selection coefficients using fitdadi (Selection module)
```{python, eval=F, python.reticulate=F}

# compute Nanc for DFE rescaling
mu = 1.5e-8
Ls = 8058343 # length 
Nanc = theta_s/(4*mu*Ls)

# compute nonsynonymous theta
mut_ratio = 2.31 # mu_ns/mu_s
theta_ns = theta_s * mut_ratio

# Demographic model with selection, see if you can find the difference
def eur_demog_sel(params, ns, pts):
	"""
	Demographic model with selection
	"""
	dadi.Integration.timescale_factor = 0.001
	N1, T1, N2, T2, NC, TC, gamma = params
	xx = dadi.Numerics.default_grid(pts)
	phi = dadi.PhiManip.phi_1D(xx, gamma=gamma, h=0.5) # dominance coefficient (assuming that mutations are additive)
	phi = dadi.Integration.one_pop(phi, xx, T1, N1, gamma=gamma) # assuming that non-synonymous mutations are gamma distributed
	phi = dadi.Integration.one_pop(phi, xx, T2, N2, gamma=gamma)
	dadi.Integration.timescale_factor = 0.000001
	nu_func = lambda t: N2*numpy.exp(numpy.log(NC/N2)*t/TC)
	phi = dadi.Integration.one_pop(phi, xx, TC, nu_func, gamma=gamma)
	fs = dadi.Spectrum.from_phi(phi, ns, (xx,))
	return fs

# set int_breaks
# assume everything >0.25 is not seen in the sample
int_breaks=[0.1, 1e-2, 1e-3, 1e-4, 1e-5]

# convert to gamma scale
int_breaks = [x*2*Nanc for x in int_breaks]
s1, s2, s3, s4, s5 = int_breaks

# make sure this is tiny relative to gamma -- minimize trapezoid integration error
eps = 1e-5
# do a custom vector of gammas
# sloppy code  
gamma_list = []
gamma_list = numpy.append(gamma_list, -numpy.logspace(numpy.log10(s1),numpy.log10(s2), 50))
gamma_list = numpy.append(gamma_list, -numpy.logspace(numpy.log10(s2-eps),numpy.log10(s3), 50))
gamma_list = numpy.append(gamma_list, -numpy.logspace(numpy.log10(s3-eps),numpy.log10(s4), 50))
gamma_list = numpy.append(gamma_list, -numpy.logspace(numpy.log10(s4-eps),numpy.log10(s5), 50))
gamma_list = numpy.append(gamma_list, -numpy.logspace(numpy.log10(s5-eps),numpy.log10(eps), 50))

print("This is the gamma list")
print(gamma_list)
# parameters for building selection spectra
ns = syn_sfs.sample_sizes

# copy Selection.py to working dir before running this code
# this takes a while
spectra = Selection.spectra(popt, ns, eur_demog_sel, pts_l=pts_l, 
							gamma_list=gamma_list, echo=True, mp=False, n=Nanc) #, cpus=30)
# you may receive a warning
# WARNING:Numerics:Extrapolation may have failed. Check resulting frequency 
# spectrum for unexpected results.
# 
# this results from small number numerical errors. you can check the most
# deleterious SFS (spectra.spectra[0]) to make sure there are no problematic
# values (i.e. large negative numbers). it's ok if there are negative numbers
# ~0

#manually add lethal bound (|s|=1)
#it's ok to do this manually because the difference between SFS
#bins between your biggest gamma and s=1 is usually really small
# so a linear approximation connecting
#SFS entries for the two gammas is fine. however make sure gamma is 
#big enough such that this is true say around ~2500 for sample size ~1000
#spectra.spectra = numpy.array([[0]*865, *spectra.spectra])
#spectra.gammas = numpy.append(-1*2*Nanc, spectra.gammas)

#pickle spectra object to use later so you don't have to wait for spectra
#object generation step in subsequent runs
#print(spectra)


#pickle.dump(spectra, open('spectra_h_{coef}.sp'.format(coef=h_coefficient),'wb'))


spectra=pickle.load(open('spectra_0.5.sp','rb'))
print(spectra)

# load nonsynonymous SFS
infilename = "1kG_nonsyn.csv"
nonsyn_sfs = numpy.genfromtxt(infilename,delimiter=",")
nonsyn_sfs = dadi.Spectrum(nonsyn_sfs)
nonsyn_sfs = nonsyn_sfs.fold()

# fit a gamma DFE
dfe=Selection.gamma_dist
lower_bound=[1e-3, 1]
upper_bound=[100,100000]
params = (0.2, 1000)

# Initiating with a random prior:
p0 = dadi.Misc.perturb_params(params, upper_bound=upper_bound)

# Optmizing your parameters given the data (nonsyn_sfs) 
popt = Selection.optimize_log(p0, nonsyn_sfs, spectra.integrate, dfe,
								 theta_ns, lower_bound=lower_bound, 
								 upper_bound=upper_bound, verbose=len(p0),
								 maxiter=25)

print(popt)
```

If this runs correctly, you should infer something close to, but not exactly, the true DFE. The results will be stored in popt. Print your results and see what you get.

The mean selection coefficient that you guys use in your slim simulations need to be re-scaled as:

E[s] = -shape * scale * 2/(2Na) = (-0.18 * 905 * 2 )/(2*12378)
E[s] = -0.01316045

The expected SFS at the MLE can be computed with:
```{python, eval =F, python.reticulate=F}
modelsfs = spectra.integrate( popt[1], Selection.gamma_dist, theta_ns)
```

You can plot your SFS in R. You can also take the gamma coefficients (scale and shape) in order to compute the probability mass for a given selection coefficient. We can create a function in R, that will both estimate the proportions in each bin while also plotting it. 

```{R, eval =F}
library(ggplot2)

plot_discrete_DFE <- function(alpha, beta, Nanc){
  # compute bin masses  
  x0 = 0. * 2 * Nanc
  x1 = 1e-5 * 2 * Nanc
  x2 = 1e-4 * 2 * Nanc
  x3 = 1e-3 * 2 * Nanc
  x4 = 1e-2 * 2 * Nanc
  x5 = 1 * 2 * Nanc
  a = pgamma(x1,shape=alpha,scale=beta)
  b = pgamma(x2,shape=alpha,scale=beta) - pgamma(x1,shape=alpha,scale=beta)
  c = pgamma(x3,shape=alpha,scale=beta) - pgamma(x2,shape=alpha,scale=beta)
  d = pgamma(x4,shape=alpha,scale=beta) - pgamma(x3,shape=alpha,scale=beta)
  e = 1 - pgamma(x4,shape=alpha,scale=beta)
  
  two_Nanc_s_range = c("0<= |s| < 10^-5", "10^-5 <= |s| <= 10^-4", 
            "10^-4 <= |s| < 10^-3", "10^-3 <= |s| <= 10^-2", "10^-2 <= |s|")
  df = as.data.frame(cbind(two_Nanc_s_range, c(a,b,c,d,e)))
  colnames(df) <- c("s", "Probability_mass")
  df$Probability_mass = as.numeric(df$Probability_mass)
  level_order <- two_Nanc_s_range
  print(sum(a,b,c,d,e))
  print(df)
  plot1 <- ggplot(data=df, aes(x=factor(s,two_Nanc_s_range), y=Probability_mass)) + geom_bar(stat="identity") + theme_bw() 
  return(plot1)
}

```

Try some of the values found within the Kim 2017 paper [here](https://www.genetics.org/content/206/1/345) to see if it agrees with their results. 

Does it work?