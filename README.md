<p align="left"> 
<img width=15% src="https://dai.lids.mit.edu/wp-content/uploads/2018/06/Logo_DAI_highres.png" alt=“Copulas” />
  <i>An open source project from Data to AI Lab at MIT.</i>
</p>

# Reversible Data Transforms

[![][pypi-img]][pypi-url] [![][travis-img]][travis-url]

- Free software: MIT license
- Documentation: https://HDI-Project.github.io/RDT

[travis-img]: https://travis-ci.org/HDI-Project/RDT.svg?branch=master
[travis-url]: https://travis-ci.org/HDI-Project/RDT
[pypi-img]: https://img.shields.io/pypi/v/RDT.svg
[pypi-url]: https://pypi.python.org/pypi/RDT


## Overview

This a python library used to transform data for data science libraries and preserve the transformations in order to reverse them as needed.

## Install

### Requirements

**RDT** has been developed and tested on [Python 3.5, 3.6 and 3.7][python-downloads]

Also, although it is not strictly required, the usage of a [virtualenv][virtualenv] is highly recommended in order to avoid interfering with other software installed in the system where **RDT** is fun.

These are the minimum commands needed to create a virtualenv using python3.6 for **RDT**:

	pip install virtualenv
	virtualenv -p $(which python3.6) rdt-venv

Afterwards, you have to execute this command to have the virtualenv activated:

	source rdt-venv/bin/activate
	
Remember about executing it every time you start a new console to work on **RDT**!

[python-downloads]: https://www.python.org/downloads
[virtualenv]: https://virtualenv.pypa.io/en/latest/

### Install with pip

After creating the virtualenv and activating it, we recommend using [pip][pip] in order to install **RDT**:

	pip install rdt

This will pull and install the latest stable release from [PyPi][pypi].

[pip]: https://pip.pypa.io/en/stable/
[pypi]: https://pypi.org/

### Install from sources

Alternatively, with your virtualenv activated, you can clone the repository and install it from source by running `make install` on the `stable` branch:

	git clone https://github.com/HDI-Project/RDT
	cd RDT
	git checkout stable
	make install

### Install for development

If you want to contribute to the project, a few more steps are required to make the project ready for development.

First, please head to the GitHub page of the project and make a fork of the project under you own username by clicking on the **fork** button on the upper right corner of the page.

Afterwards, clone your fork and create a branch from master with a descriptive name that includes the number of issues that you are going to work on:

	git clone https://github.com/HDI-Project/RDT
	cd RDT
	git branch issue-xx-cool-new-feature master
	git checkout issue-xx-cool-new-feature

Finally, install the project with the following command, wich will install some additional dependencies for code linting and testing.

	make install

Make sure to use them regularly while developing by running the commands `make lint` and `make test`.

## Quickstart

This library is used to apply desired transformations to individual tables or entire datasets
all at once, with the goal of getting completely numeric tables as the output. The desired
transformations can be specified at the column level, or dataset level. For example, you can
apply a datetime transformation to only select columns, or you can specify that you want every
datetime column in the dataset to go through that transformation.

To run the examples, we need to decompress the demo data included in the repository by running this
command on a shell:

	tar -xvzf examples/data/airbnb.tar.gz -C examples/data/

### Transforming a column

The base class of this library is the BaseTransformer class. This class provides method to fit
a transformer to your data and transform it, a method to transform new data with an already
fitted transformer and a method to reverse a transform and get data that looks like the original
input. Each transformer class inherits from the BaseTransformer class, and thus has all
these methods.

Transformers take in a column and the meta data for that column as an input. Below we will
demonstrate how to use a datetime transformer to transform and reverse transform a column.

```python
>>> from rdt.transformers import get_col_info
>>> demo_data = 'examples/data/airbnb/Airbnb_demo_meta.json'
>>> column, column_metadata = get_col_info('users', 'date_account_created', demo_data)
>>> column.head(5)
```

```
0    2014-01-01
1    2014-01-01
2    2014-01-01
3    2014-01-01
4    2014-01-01
Name: date_account_created, dtype: object
```

```python
>>> column_metadata
```

```
{'name': 'date_account_created',
 'type': 'datetime',
 'format': '%Y-%m-%d',
 'uniques': 1634}
```

Now we can transform the column.

```python
>>> from rdt.transformers import DTTransformer
>>> transformer = DTTransformer(column_metadata)
>>> transformed_data = transformer.fit_transform(column.to_frame())
>>> transformed_data.head(5)
```

```
0                      1          1.388531e+18
1                      1          1.388531e+18
2                      1          1.388531e+18
3                      1          1.388531e+18
4                      1          1.388531e+18
```

If you want to reverse the transformation and get the original data back, you can run the
following command.

```python
>>> reverse_transformed = transformer.reverse_transform(transformed_data)
>>> reverse_transformed.head(5)
```

```
  date_account_created
0           2014-01-01
1           2014-01-01
2           2014-01-01
3           2014-01-01
4           2014-01-01
```

### Transforming a table

You can also transform an entire table using the HyperTransformer class. Again, we can start by
loading the data.

```python
>>> import json
>>> from rdt.transformers import load_data_table
>>> meta_file = 'examples/data/airbnb/Airbnb_demo_meta.json'
>>> with open(meta_file, "r") as f:
...     meta_content = json.loads(f.read())
>>> table_tuple = load_data_table('users', meta_file, meta_content)
>>> table, table_meta = table_tuple
```

Now you can pass a list of the desired transformers into the `fit_transform_table` function to
transform the whole table.

```python
>>> from rdt.hyper_transformer import HyperTransformer
>>> ht = HyperTransformer(meta_file)
>>> tl = ['DTTransformer', 'NumberTransformer', 'CatTransformer']
>>> transformed = ht.fit_transform_table(table, table_meta, transformer_list=tl)
>>> transformed.head(3).T
```

```
                                     0             1             2
?date_account_created     1.000000e+00  1.000000e+00  1.000000e+00
date_account_created      1.388531e+18  1.388531e+18  1.388531e+18
?timestamp_first_active   1.000000e+00  1.000000e+00  1.000000e+00
timestamp_first_active    1.654000e+13  1.654000e+13  1.654000e+13
?date_first_booking       1.000000e+00  0.000000e+00  0.000000e+00
date_first_booking        1.388790e+18  0.000000e+00  0.000000e+00
?gender                   1.000000e+00  1.000000e+00  1.000000e+00
gender                    8.522112e-01  3.412078e-01  1.408864e-01
?age                      1.000000e+00  0.000000e+00  0.000000e+00
age                       6.200000e+01  3.700000e+01  3.700000e+01
?signup_method            1.000000e+00  1.000000e+00  1.000000e+00
signup_method             3.282037e-01  3.500181e-01  4.183867e-01
?signup_flow              1.000000e+00  1.000000e+00  1.000000e+00
signup_flow               4.453093e-01  3.716032e-01  3.906801e-01
?language                 1.000000e+00  1.000000e+00  1.000000e+00
language                  2.927157e-01  5.682538e-01  6.622744e-01
?affiliate_channel        1.000000e+00  1.000000e+00  1.000000e+00
affiliate_channel         9.266169e-01  5.640470e-01  8.044208e-01
?affiliate_provider       1.000000e+00  1.000000e+00  1.000000e+00
affiliate_provider        7.717574e-01  2.539509e-01  7.288847e-01
?first_affiliate_tracked  1.000000e+00  1.000000e+00  1.000000e+00
first_affiliate_tracked   3.861429e-01  8.600605e-01  4.029200e-01
?signup_app               1.000000e+00  1.000000e+00  1.000000e+00
signup_app                6.915504e-01  6.373492e-01  5.798949e-01
?first_device_type        1.000000e+00  1.000000e+00  1.000000e+00
first_device_type         6.271052e-01  2.611754e-01  6.828802e-01
?first_browser            1.000000e+00  1.000000e+00  1.000000e+00
first_browser             2.481743e-01  5.087636e-01  5.023412e-01
```

You can then reverse transform the output to get a table in the original format, but it will
only contain the columns corresponding to those that were transformed (ie. numeric columns).

```python
>>> reverse_transformed = ht.reverse_transform_table(transformed, table_meta)
>>> reverse_transformed.head(3).T
```

```
                                       0               1                2
date_account_created          2014-01-01      2014-01-01       2014-01-01
timestamp_first_active    19700101053540  19700101053540   19700101053540
date_first_booking            2014-01-04             NaN              NaN
gender                              MALE       -unknown-        -unknown-
age                                   62             NaN              NaN
signup_method                      basic           basic            basic
signup_flow                            0               0                0
language                              en              en               en
affiliate_channel          sem-non-brand          direct        sem-brand
affiliate_provider                google          direct           google
first_affiliate_tracked              omg       untracked              omg
signup_app                           Web             Web              Web
first_device_type        Windows Desktop     Mac Desktop  Windows Desktop
first_browser                     Chrome         Firefox          Firefox
```

### Transforming a dataset

The hyper transformer is also capable of transforming all of the tables specified in your
meta.json at once.

```python
>>> from rdt.hyper_transformer import HyperTransformer
>>> meta_file = 'examples/data/airbnb/Airbnb_demo_meta.json'
>>> ht = HyperTransformer(meta_file)
>>> tl = ['DTTransformer', 'NumberTransformer', 'CatTransformer']
>>> transformed = ht.fit_transform(transformer_list=tl)
>>> transformed['users'].head(3).T
```

```
                                     0             1             2
?date_account_created     1.000000e+00  1.000000e+00  1.000000e+00
date_account_created      1.388531e+18  1.388531e+18  1.388531e+18
?timestamp_first_active   1.000000e+00  1.000000e+00  1.000000e+00
timestamp_first_active    1.654000e+13  1.654000e+13  1.654000e+13
?date_first_booking       1.000000e+00  0.000000e+00  0.000000e+00
date_first_booking        1.388790e+18  0.000000e+00  0.000000e+00
?gender                   1.000000e+00  1.000000e+00  1.000000e+00
gender                    9.061832e-01  1.729590e-01  4.287514e-02
?age                      1.000000e+00  0.000000e+00  0.000000e+00
age                       6.200000e+01  3.700000e+01  3.700000e+01
?signup_method            1.000000e+00  1.000000e+00  1.000000e+00
signup_method             5.306912e-01  4.082081e-01  3.028973e-01
?signup_flow              1.000000e+00  1.000000e+00  1.000000e+00
signup_flow               4.597129e-01  4.751324e-01  5.495054e-01
?language                 1.000000e+00  1.000000e+00  1.000000e+00
language                  2.947847e-01  4.170684e-01  5.057820e-01
?affiliate_channel        1.000000e+00  1.000000e+00  1.000000e+00
affiliate_channel         9.213130e-01  4.712533e-01  8.231925e-01
?affiliate_provider       1.000000e+00  1.000000e+00  1.000000e+00
affiliate_provider        7.649791e-01  2.028804e-01  7.174262e-01
?first_affiliate_tracked  1.000000e+00  1.000000e+00  1.000000e+00
first_affiliate_tracked   3.716114e-01  6.723371e-01  3.710109e-01
?signup_app               1.000000e+00  1.000000e+00  1.000000e+00
signup_app                3.583918e-01  2.627690e-01  4.544640e-01
?first_device_type        1.000000e+00  1.000000e+00  1.000000e+00
first_device_type         6.621950e-01  3.078130e-01  7.152115e-01
?first_browser            1.000000e+00  1.000000e+00  1.000000e+00
first_browser             2.410379e-01  4.766930e-01  4.865389e-01
```

```python
>>> transformed['sessions'].head(3).T
```

```
                         0             1           2
?action           1.000000      1.000000    1.000000
action            0.361382      0.597891    0.353806
?action_type      1.000000      1.000000    1.000000
action_type       0.089913      0.560351    0.046400
?action_detail    1.000000      1.000000    1.000000
action_detail     0.070212      0.852246    0.107477
?device_type      1.000000      1.000000    1.000000
device_type       0.726447      0.711231    0.710298
?secs_elapsed     1.000000      1.000000    1.000000
secs_elapsed    319.000000  67753.000000  301.000000
```

```python
>>> reverse_transformed = ht.reverse_transform(tables=transformed)
>>> reverse_transformed['users'].head(3).T
```

```
                                       0               1                2
date_account_created          2014-01-01      2014-01-01       2014-01-01
timestamp_first_active    19700101053540  19700101053540   19700101053540
date_first_booking            2014-01-04             NaN              NaN
gender                              MALE       -unknown-        -unknown-
age                                   62             NaN              NaN
signup_method                      basic           basic            basic
signup_flow                            0               0                0
language                              en              en               en
affiliate_channel          sem-non-brand          direct        sem-brand
affiliate_provider                google          direct           google
first_affiliate_tracked              omg       untracked              omg
signup_app                           Web             Web              Web
first_device_type        Windows Desktop     Mac Desktop  Windows Desktop
first_browser                     Chrome         Firefox          Firefox
```

```pythnon
>>> reverse_transformed['sessions'].head(3).T
```

```
                             0                    1                2
action                  lookup       search_results           lookup
action_type               None                click             None
action_detail             None  view_search_results             None
device_type    Windows Desktop      Windows Desktop  Windows Desktop
secs_elapsed               319                67753              301
```
