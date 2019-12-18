## Description
Python implementation of the PUMA algorithm.

## Table of Contents
* [Links to literature](#links-to-literature)
* [Puma algorithm](#puma-algorithm)  
* [Installation](#installation)  
* [Usage](#usage)  
  * [Run from terminal](#run-from-terminal)
  * [Run from python](#run-from-python)
* [Toy data](#toy-data)
* [Results](#results)


## Links to literature 

* **PUMA** (PANDA Using MicroRNA Associations)  
_Manuscript in preparation, used in [Hill et al.](https://jhoonline.biomedcentral.com/articles/10.1186/s13045-017-0465-4)._  
C and MATLAB code: [https://github.com/mararie/PUMA](https://github.com/mararie/PUMA)

* **[PANDA](http://journals.plos.org/plosone/article?id=10.1371/journal.pone.0064832)** (Passing Attributes between Networks for Data Assimilation)
_Glass K, Huttenhower C, Quackenbush J, Yuan GC. Passing Messages Between Biological Networks to Refine Predicted Interactions, PLoS One, 2013 May 31;8(5):e64832_  
Original PANDA C++ code: [http://sourceforge.net/projects/panda-net/](http://sourceforge.net/projects/panda-net/).  

* **[LIONESS](https://www.sciencedirect.com/science/article/pii/S2589004219300872)** (Linear Interpolation to Obtain Network Estimates for Single Samples)
_Kuijjer ML, Tung M, Yuan GC, Quackenbush J, Glass K. Estimating sample-specific regulatory networks, iScience, 2019 Apr 26;14:226-240_

## The PUMA algorithm
PUMA starts with a prior network of putative regulatory interactions (center network in the image below), a prior network of protein-protein interactions between transcription factors (optional input), and target gene expression data, which is converted into a co-expression network.

<img src="img/puma.png" height="300">  

A message passing framework is used to find agreement between the three input networks. First, the responsibility (R) is calculated:

<img src="img/responsibility.png" height="30"> ,

as well as the availability (A):

<img src="img/availability.png" height="30">.

The prior gene regulatory network W is then updated using the responsibility and availability:

<img src="img/combine.png" height="30">.

Next, the protein cooperativity and gene co-regulatory networks are updated:

<img src="img/cooperativity.png" height="100">  
<img src="/img/co-regulatory.png" height="30">

, while self-interactions in P and C are updated to satisfy convergence:

<img src="img/p.png" height="30">  
<img src="/img/c.png" height="30">

, which is evaluated using a hamming distance:

<img src="img/hamming.png" height="52">.


## Installation
PyPuma runs on Python 3.x. You can either run the PyPuma script directly (see [Usage](#usage)) or install it on your system. We recommend the following commands to install PyPuma on UNIX systems:
#### Using  a virtual environment
Using [python virtual environments](http://docs.python-guide.org/en/latest/dev/virtualenvs/) is the cleanest installation method. 

Cloning git and setting up a [python virtual environment](http://docs.python-guide.org/en/latest/dev/virtualenvs/):
```no-highlight
pip install --user pipenv   #Make sure you have pipenv
git clone https://github.com/aless80/PyPuma.git
cd PyPuma
```
Creating a virtual environment and installing PyPuma:
```no-highlight
virtualenv pypumaenv #virtual environment created in a folder inside the git folder 
source pypumaenv/bin/activate
(pypumaenv)$ pip install -r requirements.txt
(pypumaenv)$ python setup.py install --record files.txt
```
Uninstall PyPuma from virtual environment:
```no-highlight
cat files.txt | xargs rm -rf
```
Complete removal of virtual environment and PyPuma:
```no-highlight
(pypuma)$ deactivate	#Quit virtual environment
rm -rf pypumaenv
```

#### Using pip 
Never use ~~sudo pip~~. Instead you can use pip on the user's install directory:
```no-highlight
git clone https://github.com/aless80/PyPuma.git
cd PyPuma
python setup.py install --user
#to run from the command line you will need to make PyPuma executable and add the bin directory to your PATH:
cd bin
chmod +x PyPuma
echo "$(pwd):PATH" >> ~/.bashrc
source ~/.bashrc
```
To run PyPuma from Windows (not fully tested) install Git (https://git-scm.com/downloads) and Anaconda (https://www.continuum.io/downloads) and from the Anaconda prompt run:
```no-highlight
git clone https://github.com/aless80/PyPuma.git
cd PyPuma
python setup.py install
```

## Usage
#### Run from terminal
PyPuma can be run directly from the terminal with the following options:
```
-h help
-e, --expression: expression values
-m, --motif: pair file of motif edges, or Pearson correlation matrix when not provided 
-p, --ppi: pair file of PPI edges
-o, --output: output file
-i, --mir: mir data miR file
-r, --rm_missing
-q, --lioness: output for Lioness single sample networks 
```
To run PyPuma on the included Toy example:
```
python run_puma.py -e ./ToyData/ToyExpressionData.txt -m ./ToyData/ToyMotifData.txt -p ./ToyData/ToyPPIData.txt -i ToyData/ToyMiRList.txt -o output_puma.txt
```
To run LIONESS on PUMA networks, use the flag -q (note that this can take a long time and use considerable computing resources):

```python
python run_puma.py -e ./ToyData/ToyExpressionData.txt -m ./ToyData/ToyMotifData.txt -p ./ToyData/ToyPPIData.txt -i ToyData/ToyMiRList.txt -o output_puma.txt -q output_lioness.txt
```
Finally, note that running PUMA without importing motif and expression data will estimate a co-expression network using Pearson correlation.

#### Run from python
Fire up your python shell or ipython notebook. 
Import the classes in the PyPuma library:
```python
from pypuma.puma import Puma
from pypuma.lioness import Lioness
```
Then run PUMA:
```python
puma_obj = Puma('ToyData/ToyExpressionData.txt', 'ToyData/ToyMotifData.txt', 'ToyData/ToyPPIData.txt','ToyData/ToyMiRList.txt')
```
Save the results:
```python
puma_obj.save_puma_results('Toy_Puma.pairs.txt')
```
Example of returning a network visualization of the top edges:

```python
puma_obj.top_network_plot(top=70, file='top_genes.png')
```
<!--
or
```python
from PyPuma.analyze_puma import AnalyzePuma
plot = AnalyzePuma(puma_obj)
plot.top_network_plot(top=100, file='top_100_genes.png')
```-->
Calculate indegrees for further analysis:
```python
indegree = puma_obj.return_puma_indegree()
```

Calculate outdegrees for further analysis:
```python
outdegree = puma_obj.return_puma_outdegree()
```
Run the LIONESS algorithm for single sample networks:
```python
lioness_obj = Lioness(puma_obj)
```
Save LIONESS results:
```python
lioness_obj.save_lioness_results('Toy_Lioness.txt')
```
Return a network plot for one of the LIONESS single sample networks:
```python
plot = AnalyzeLioness(lioness_obj)
plot.top_network_plot(column= 0, top=100, file='top_100_genes.png')
```

## Toy data
The example gene expression data that we have available here contains gene expression profiles for different samples in the columns. Of note, this is just a small subset of a larger gene expression dataset. We provided these "toy" data so that the user can test the method. 

However, if you plan to model gene regulatory networks on your own dataset, you should use your own expression data as input.
