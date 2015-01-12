AdFisher
=========

AdFisher is a tool for running Automated Experiments on Personalized Ad Settings. 

Requirements
-----------
AdFisher runs only on UNIX environments. It uses some standard packages listed here. 
The commands provided for installation work on Ubuntu and OS X. You may find it useful to install packages using `pip`. 
You can install `pip` by following the instructions provided [here](http://pip.readthedocs.org/en/latest/installing.html).
In order to run experiments for data collection, you will need the following packages:

  - selenium```sudo pip install selenium```
  - xvfb ```sudo apt-get install xvfb```
  - xvfbwrapper ```sudo pip install xvfbwrapper```

Selenium is a web-browser automation framework. Xvfb allows for headless testing. 
xvfbwrapper is a python wrapper for the same. 
The Xvfb package is not present on OS X, but you still have to install xvfbwrapper.
To carry out the data analysis, you require the following packages:
  - numpy, scipy, matplotlib ```sudo pip install numpy scipy matplotlib```*
  - scikit learn ```sudo pip install scikit-learn```
  - stemming ```sudo pip install stemming```
  - nltk ```sudo pip install -U pyyaml nltk```
     - You also need to download the nltk stopwords corpus by typing the following commands in your python interpreter. 
```python
import nltk
nltk.download()
``` 
pip* sometimes cannot install numpy, scipy, matplotlib on Ubuntu. In that case, run 
```sudo apt-get install python-numpy python-scipy python-matplotlib```.
NumPy and SciPy are Python packages for scientific computing. matplotlib enables plotting functions. 
scikit learn has a vast collection of python implemenations of Machine Learning algorithms, 
built on the NumPy, SciPy, and matplotlib packages. 
We use the stemming package to stem words, and the nltk stopwords corpus for identifying stopwords.

Example
-----------
An example piece of code to run the experiment and perform the analysis looks as follows. 
This script must be run within the AdFisher directory - the one which also has the `core` folder.
```python
import core.adfisher as adfisher

site_file = 'employment.txt'

# Collect sites from alexa

adfisher.collect_sites_from_alexa(nsites=5, output_file=site_file,
	alexa_link="http://www.alexa.com/topsites/category/Top/Business/Employment")

# Set up treatments

treatment1 = adfisher.Treatment("female")
treatment1.opt_in()
treatment1.set_gender("female")
treatment1.visit_sites(site_file)

treatment2 = adfisher.Treatment("male")
treatment2.opt_in()
treatment2.set_gender("male")
treatment2.visit_sites(site_file)

## Set up measurement

measurement = adfisher.Measurement()
measurement.get_age()
measurement.get_gender()
measurement.get_language()
measurement.get_interests()
measurement.get_ads(site="toi", reloads=10, delay=5)

## Run Experiment

adfisher.run_experiment(treatments=[treatment1, treatment2], measurement=measurement, 
	agents=10, blocks=100, log_file=log_file)
# Analyze Data

adfisher.run_ml_analysis()

```

Now, we explain each part of the above script in detail.
In order to run an experiment, you first need to import the adfisher module.
```python
import core.adfisher as adfisher
```
### Setting up treatments
You can initialize a treatment with its name as follows.
```python
treatment = adfisher.Treatment("name")
```
##### Google Ad Settings
The default treatment is equivalent to the null treatment. Browser instances initiated with this treatment enable behavioral advertisements in the treatment phase.
We provide several actions that can be performed as part of the treatment. You can opt in to or opt out of seeingg behavioral ads by Google across the web.
You may also set your gender, age group, or interests on Google Ad Settings as part of the treatment.
```python
treatment.opt_in()
treatment.set_gender("female")
treatment.set_age(22)
treatment.add_interest("Autos & Vehicles")
treatment.remove_interest("Basketball")
treatment.opt_out()
```
There are few things one must be careful about here. None of the settings are available on the Google Ad Settings page unless one 'opts in' first. Also, there is no setting for a specific age, but only for an age group. AdFisher sets the corresponding age group when given an age. Upon searching for an interest to add, Google provides a range of options when available. AdFisher will only add the first interest prompted by Google. However, when removing an interest, AdFisher will delete all interests that contain the keyword. For example ```remove_interest("Basketball")``` will remove both *Basketball* and *Basketball Equipment*, if present.
##### Visit Sites
AdFisher also allows you to visit websites as part of the treatment.By passing the name of a file containing the URLs of websites, AdFisher will propel the corresponding agents to visit all the websites one after the other.
```python
treatment.visit_sites(site_file)
```
In case you would like to collect the names of websites associated with a particular [category on Alexa](http://www.alexa.com/topsites/category/Top), AdFisher launches a signle browser instance to collect the URLs into a file. For example, the following collects the top 100 websites on Alexa for employment and saves them in *employment.txt*
```python
adfisher.collect_sites_from_alexa(output_file="employment.txt", nsites=100, browser="firefox",
        alexa_link="http://www.alexa.com/topsites/category/Top/Business/Employment")
```
### Setting up measurement
AdFisher gives you the option to specify what measurements you want to collect on the browser agents after the completion of application of respective treatments. Initialize a measurement using the following statement.
```python
measurement = adfisher.Measurement()
```
AdFisher currently allows you to read the age, gender, language, and interests set on Google's Ad Settings, and collect advertisments from "toi" (Times of India), "bbc", "guardian", "reuters" or "bloomberg". You can also set the number of reloads of a news websites, or the delay between successive reloads.

```python
measurement.get_age()
measurement.get_gender()
measurement.get_language()
measurement.get_interests()
measurement.get_ads(site="toi", reloads=10, delay=5)
```

### Run Experiment
Once the treatments and measurement are set up, you are ready to run the experiment. You must at least specify the list of treatments in order to run the experiment; all other parameters have default values. The *log_file* maintains a log of the experiments. All data from the experiments are stored in this file in a special-character separated format, with each new line containing an entry. The *log_file* is later used to perform data analysis. 

You can specify the number of blocks of the experiment in *blocks*. *samples* specifies the number of browser instances running in a block. *runs* specifies the number of iterations of treatment and collection a single browser instance goes through. 


The experiment can be run on either "firefox" or "chrome" browsers as of now, and that can be specified with the *browser* parameter. You can set the timeout of a particular block by specifying the *timeout* parameter. By default, a block times out after 2000 seconds (~30 mins). 
```python
adfisher.run_experiment(treatments, measurement, log_file="log.txt", blocks=20, agents=2, 
        runs=1, browser="firefox", timeout=2000)	
```
### Perform Analysis

After collection, the analysis can be carried out in two modes. First, where a keyword based statistic is used to carry out the permutation test. By default, the statistic is the difference in the number of ads containing the keywords from treatment1 and the same from treatment2. Since this is a one-sided test, you can also test the other end of the spectrum by simply setting the *flipped* argument to True. *log_file* specifies the log file from the experiment. 

```python
adfisher.run_kw_analysis(log_file="log.txt", keywords=['rehab'], flipped=False, verbose=False)
```

However, if you choose to use Machine Learning to choose your test statistic, you can run the analysis in the second mode.
*splitfrac* indicates the splitting fraction for the training and testing data. If it is set to 0.1, the first 90% of the blocks are used for training, and the last 10% are used for testing. The number of folds in k-fold cross validation is specified in *nfolds*. *blocks* must be at least as large as *nfolds*. 

*feat_choice* can be either "ads" or "words". It specifies what to the classifier uses as features. The number of features output from the feature selection algorithm is given by *nfeat*. 
```python
adfisher.run_ml_analysis(log_file="log.txt", splitfrac=0.1, nfolds=10, 
		feat_choice="ads", nfeat=5, verbose=False)
```
This calls a sequence of analysis techniques. First, the parameteres of classifier is selected by running k-fold cross validation. Then this classifier is used to generate statistics to run a permutation test, which is a statistical test for significance. Finally, the top features are printed out for each of the treatments. 

