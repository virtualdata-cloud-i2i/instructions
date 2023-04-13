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

## User-based code

For the sake of the exercises, we will play with a simple user-based code (that could be your code in the future!). On your VM, simply clone the repository:

```bash
git clone git@github.com:virtualdata-cloud-i2i/myapp.git
```

and execute:

```bash
cd myapp
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

## Example 2: building a microservice

Often, the difficulty for someone else to use our codes lies around the code: installation headache, environment conflict hell, migration nightmare, and so on. But the cloud can help us! Why not installing our software on a VM, and then exposing its functionalities via simple protocol?

This is the basis of microservices for examples. Our code will run in an isolated environment that we control, and users will communication with the code via API calls over the web. An API, or application programming interface, is a set of rules that define how applications or devices can connect to and communicate with each other. For the web, one of the most popular API is the REST API, which is an API that conforms to the design principles of the [REST](https://en.wikipedia.org/wiki/Representational_state_transfer), or representational state transfer software architectural style. I am sure -- no I am ready to bet billions -- that you manipulate REST API everyday without even knowning it!

We include a REST API alongside our web server from exercice 1. For example, we can construct a simple microservice around our function `mylib.add_numbers`

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

Uh ?! what is happening here?


## Example 3: Continuous integration and coverage

Going further, we can also use the cloud to deal with borring and lengthy operations, and only expose a nice summary to the users. Let's take the example of testing your code (see the amazing lecture: https://github.com/JulienPeloton/robprog). Test suites (if any!) can be long, and we often put it off. Fortunately, there are many possibilities now to _continuously test and deploy_ our applications via the famous CI/CD components of our favourite forges (GitHub Actions, GitLab CI, etc.). But have thought what is really behind those?

Let's take GitHub. Thanks to Mr Microsoft, there is the cloud Azure in the backend. Anytime you trigger a CI job (REST API!), a VM is allocated, a container is spawned, your environment is installed, your code is tested, and test results are sent for further inspection (REST API!). So basically you need 4 things: a VM to host the web server for your frontend operations, a VM to execute the code, another VM to deploy a database to store all products, and finally a VM to host the REST server which handles all communications. Well in practice this might be slightly more complicated, but that's the idea.

Here I propose to simply do everything on the same VM, just to experiment a bit the different components. So we will have to adapt a bit our current code. First notice that our web application has an endpoint to trigger the test suite of our simple code. In the terminal of your laptop for example:

```bash
IP=...
PORT=...
curl -H "Content-Type: application/json" -X POST \
    -d '{"branch": "main"}' \
    http://$IP:$PORT/api/v1/ci
```

So what happened? You communicate with the server with the REST API. You told the server to take the `myapp` code on branch main, to run the test suite, and to send back the result of the tests. As simple as that! Try also with a failure:

```bash
IP=...
PORT=...
curl -H "Content-Type: application/json" -X POST \
    -d '{"branch": "ci_fail"}' \
    http://$IP:$PORT/api/v1/ci
```

Then, in a browser open:

```
http://IP:PORT/static/main/index.html
```

to inspect the coverage. Exercise: make it more friendly!

## Example 4: implement a streaming application