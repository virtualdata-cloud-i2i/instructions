# Cloud computing with VirtualData

Materials for the Cloud Computing lecture -- Informatique des 2 infinis

This crash course focuses on cloud computing, and especially the use of VirtualData in a scientific context. The examples are mainly based on the Python programming language, but it is meant to be general.

## First steps with VirtualData

You have two ways to interact with the cloud:
- Command line interface: [tutorial](https://gist.github.com/JulienPeloton/bb77476623a090c60ee1b7c2a2791699)
- Web-based dashboard: [link](https://keystone.lal.in2p3.fr)

Make sure you can request resources before trying the examples.

## Before starting

Make sure you can connect to your machine:

```bash
ssh -i ~/.ssh/twoinfinities centos@xx.xx.xx.xx
```

the key `twoinfinities` should have permission only for the user (otherwise ssh will yell at you):

```bash
ls -lt ~/.ssh/twoinfinities
-rw------- 1 peloton peloton 1675 Apr 13 13:34 /home/peloton/.ssh/twoinfinities
```

If this is not the case, just run:

```bash
chmod 600 ~/.ssh/twoinfinities
```

### Basic setup of the VM

Instal git:

```bash
sudo yum install git
```

and clone the repository:

```bash
git clone https://
```

Install other utilities if need be:

```bash
# e.g.
sudo yum install wget vim
```

### Miniconda installation

By default, our OS proposes only python 2 (!), so we have to install a newer Python version. As we chose a VM with only 2 vCPU, install the full Anaconda might be a killer. Instead, let's install miniconda:

```bash
# should be quick
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh

# follow instructions
bash Miniconda3-latest-Linux-x86_64.sh

# Update your shell
source ~/.bashrc

# Check python version
python --version
Python 3.10.9
```

Then install and activate the environment:

```bash
cd /path/to/virtualdata
conda env create -f environment.yml
conda activate vd
```

and install the specific application dependencies:

```bash
cd /path/to/virtualdata
pip install .
```

## Example 1: web application

Hosting web applications is a common case for clouds. Connect to your VM, and clone the repository containing a simple web application:

```bash
git clone git@github.com:virtualdata-cloud-i2i/webapp.git
```

Enter the repository, and execute:

```bash
cd /path/to/virtualdata
python3 index.py
```

And follow instructions. This application is based on [Dash](https://dash.plotly.com) which makes the server deployment super easy.

## Example 2: REST API

talk about REST.

We include a REST API alongside our web server from exercice 1. For example, we can construct a simple microservice around our function `apps.mylib.add_numbers`

In Python:

```python
import requests

IP = ...
PORT = ...

r = requests.post('http://{}:{}/api/v1/add_numbers'.format(IP, PORT), json={'num1': 1, 'num2': 1})
# <Response [200]>

r.content
# b'[{"Result":2}]'
```

or curl:

```bash
IP=...
PORT=...
curl -H "Content-Type: application/json" -X POST \
    -d '{"num1": 1.0, "num2": 1.0}' \
    http://$IP:$PORT/api/v1/add_numbers
# [{"Result":2.0}]
```

try also in your browser:

```
http://IP:PORT/api/v1/add_numbers?num1=1&num2=2
```

what is happening?


## Example 3: Continuous integration and coverage

```
./run_ci.sh --branch init

http://127.0.0.0:24000/static/init/index.html
```