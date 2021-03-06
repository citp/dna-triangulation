Identification of Anonymous DNA Using Genealogical Triangulation
===============

Consumer genetics databases hold dense genotypes of millions of
people, and the number is growing quickly. In 2018, law enforcement
agencies began using such databases to identify anonymous DNA via
long-range familial searches. We show that this technique is far more
powerful if combined with a genealogical database of the type
collected by online ancestry services. We present a "genealogical
triangulation" algorithm and study its effectiveness on simulated
datasets. We show that for over 50% of targets, their anonymous DNA
can be identified (matched to the correct individual or same-sex
sibling) when the genetic database includes just 1% of the
population. We also show the effectiveness of "snowball
identification" in which a successful identification adds to the
genetic genealogical database, increasing the identification accuracy
for future instances. We discuss our technique’s potential to enhance
law enforcement capabilities as well as its privacy risks.

https://www.biorxiv.org/content/10.1101/531269v1

Getting Started
===============

This project requires both [Rust](https://www.rust-lang.org/) and
[Python](https://www.python.org/). Populations are generated in
Python, then exported to a json file. Rust reads the population file,
then runs simulations to generate IBD data, outputting the data to a
file. The output is then read in on the python side and fit to
generate a model. Identification tasks are performed in Python using
the generated model.

Identification will require 16+GB of RAM for populations with
generations of 100k individuals.

System requirements
-----------

* Python
* Rust
* GNU scientific library (GSL)

Python Requirements
-----------

* Python 3.4+
* scipy/numpy
* Cython
* Statsmodels
* progressbar2



Setup
-----

* run `fetch.sh` in `data/recombination_rates` directory. This will
  fetch the appropriate recombination data from the HapMap project.
* Install the python dependencies listed above.
* run `python3 setup.py build_ext --inplace` in the "python" directory
  to build the cython modules. This will need to be run whenever any
  of the .pyx files are modified.

Running
=======

Some of the script names aren't descriptive of what is happening, as code changed over the course of the project.

Work flow
---------

The typical work flow is a three step process

1. Generate population - In the `python` directory - `python3
generate_populatio n.py --help`
2. Export population to Rust - In the `python` directory - `python3
   export_population.py --help`
4. Run simulations in rust - In the `rust` directory - `cargo run
   --release --bin simulate -- --help`
5. Import output from simulations - In the `python` directory -
   `python3 import_simulation.py --help`
6. Identify - In the `python` directory - `python3
   evaluate_deanonymize.py --help`

Commands
--------

### Generate population

To generate a population use the `generate_population.py` script. For
example if you cd into the `python` directory and run: `python3
generate_population.py ../data/your_tree_file ../data/recombination_rates/
--generation_size 1000 --num_generations 10 --output
population.pickle` A population with 10 generations each with 1000
members will be generated and saved to population.pickle with Python's
pickle format. In the paper the generation size was 100,000.


### Simulate population to generate distributions

To run experiments in rust first convert the population to a format the simulation can understand.: `python3
export_population.py population.pickle file_for_simulation.json --num-anchor-nodes 150`

This command will pick 150 nodes from the last three generations and mark
them as anchors. 
It will then output a json file for the rust simulator to read.
Then run `cargo run --release --bin simulate file_for_simulation.json recombination_directory output_file`
Then it will perform 1000 experiments to sample
from the simulated empirical IBD distributions for the (labeled,
unlabeled) pairs. A file `output_file` will be created with the simulation data. Simulation can run for days, depending on your population parameters and hardware. The simulations from the paper took 20-60 hours to run.

To create the model (ie fit the hurdle-gamma parameters), run `import_simulation.py population.pickle work_file --output-pickle model.pickle`. The model will be saved to `model.pickle`.

### Identify individuals

The final step is identifying individuals.

Running `python3 evaluate_deanonymize.py population.pickle
model.pickle -n 10` will try to identify 10 random unlabeled
individuals in the population.
