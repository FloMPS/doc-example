# Guide : Create a Local Conda Package 

## Introduction
We want to easily let people install a python package, and install all its dependencies automaticaly using conda.

For example let's say we created a python library named ```test_pkg```. It has a module called ```test_mod.py```.
In this module we need external dependencies like ```numpy``` and ```pandas```. 
```python
import numpy as np
import pandas as pd

...

def hello():
    print(f"Hello from test_mod.py in {__name__} package!")

...

```

Instead of publishing our code anywhere, we want to have a single local file (conda package).

Installing it in a conda environement will install numpy and pandas automatically, and make possible to call our hello function like this:
```python
import test_pkg.test_mod as tm
tm.hello()
```

# Steps
 
## 1.	Setup the structure of your project.
Following the example above, we need the following structure:
```bash
my_project
├── src
│   └── test_pkg
│       ├── __init__.py
│       └── test_mod.py
├── README.md
├── LICENSE
├── pyproject.toml
└── meta.yml
```

- ```my_project``` is the root of our project, it can be named anything.
- ```src``` folder contains the source code of our project.
- ```test_pkg``` folder will be the name of your package, this folder must contain an empty file called ```__init__.py``` file and all the modules of your package.
- ```README.md``` is the readme file of our project, not mandatory.
- ```LICENSE``` is the license file of our project, not mandatory.
- ```pyproject.toml``` is a configuration file for our project, mandatory.
- ```meta.yml``` is the configuration file of our conda package, mandatory.

## 2. Chose you LICENSE
Our example has a MIT license, but you can chose any license you want. You can find a list of licenses [here](https://choosealicense.com/). Copy-paste it into the ```LICENSE``` file.

```
MIT License

Copyright (c) [year] [fullname]

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```
Dont forget to set the ```[year]``` and ```[fullname]``` fields.

## 3. Edit the ```pyproject.toml``` file
Just copy-paste the following lines in the ```pyproject.toml``` file:
```toml
[build-system]
requires = ["setuptools>=61.0"]
build-backend = "setuptools.build_meta"
```

## 4. Edit the ```meta.yaml``` file
Copy-paste the following lines in the ```meta.yaml``` file:
```yaml
package:
  name: test_pkg_mps
  version: 0.0.1

source:
  path: .

build:
  script: {{ PYTHON }} -m pip install . -vv 
  number: 0

requirements:
  build:
    - {{ compiler('c') }}
  host:
    - python 3.10
    - pip
  run:
    - python 3.10
    - pandas
    - numpy 
    - scipy
    - pyserial
    - openpyxl
    - matplotlib

about:
  home: https://github.com/your_organization/test_pkg
  summary: 'My super python package'
  description: |
    This is a test package to show how to create a conda package.
    It has a cool hello() function that prints a message.
  # Remember to specify the license variants for BSD, Apache, GPL, and LGPL.
  # Use the SPDX identifier, e.g: GPL-2.0-only instead of GNU General Public License version 2.0
  # See https://spdx.org/licenses/
  license: MIT
  # The license_family, i.e. "BSD" if license is "BSD-3-Clause". 
  # Optional
  license_family: MIT
  # It is required to include a license file in the package,
  # (even if the license doesn't require it) using the license_file entry.
  # Please also note that some projects have multiple license files which all need to be added using a valid yaml list.
  # See https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html#license-file
  license_file: LICENSE
  # The doc_url and dev_url are optional.
  doc_url: https://test_pkg.readthedocs.io/
  dev_url: https://github.com/your_organization/test_pkg
```

### 4.1 Fields to update
- ```name```: name of your conda package, i.e. when calling ```conda install test_pkg_mps -c ....``` It is not necessary to have the same name as your python package (```test_pkg``` in our example).
- ```version```: version of your conda package.
- ```requirements: run```: list of all the dependencies of your python package. You can add any package you want, but you must have at least ```python``` and your python dependencies. Don't forget to specify the version of the libraries if you need a specific one (```numpy=1.21.2``` or ```numpy>=1.21.2``` for example)
- ```about```: This section is not mandatory, but it is a good practice to fill it. It is displayed if you upload it in the Anaconda.org channel, but has no impact if you only use it locally. If you picked and other LICENSE in step 2, you should also update the ```license``` and ```license_file``` fields.

In case you have complicated dependencies, or some that need some advanced compiler options, you might have issues later that we will not cover here. In this case, you should explore the problems on internet or in the official [conda documentation](https://docs.conda.io/projects/conda-build/en/latest/resources/define-metadata.html#requirements-section).


## 5. Build the conda package

### 5.1. Create a conda environement and build the package
Open an anaconda (miniconda) prompt :
- On Windows: press the windows key and type ```anaconda prompt```

```bash
cd C:\path\to\your\project\my_project
conda create -n my_env python=3.10 conda-build
conda activate my_env
conda build .
```
Look carefully at the output, if everything went well, you should have a line like this at about the end of the log:
```bash
...
# To have conda build upload to anaconda.org automatically, use
# $ conda config --set anaconda_upload yes
anaconda upload ^
    C:\Users\your.name\AppData\Local\anaconda3\envs\my_env\conda-bld\win-64\test_pkg_mps-0.0.1-py310hf4a77e7_0.tar.bz2
anaconda_upload is not set.  Not uploading wheels: []
...
```
This is where you can find the final conda package. Go find it in your file explorer. You can copy it and share it with anyone you want.

# 6. Install the conda package

Let's say you received the package in the following directory: ```C:\path\to\your\conda\package\test_pkg_mps-0.0.1-py310hf4a77e7_0.tar.bz2```
### 6.1. Create a conda environement and install the package
Open an anaconda (miniconda) prompt :
- On Windows: press the windows key and type ```anaconda prompt```

```bash
conda create -n my_env python=3.10 
conda activate my_env
conda install test_pkg_mps -c "C:\path\to\your\conda\package\"
``` 

And that's it! You can now use your package in your python code in your new my_env conda environement:
```python
import test_pkg.test_mod as tm
tm.hello()

# Output:
# Hello from test_mod.py in test_pkg.test_mod package!
```