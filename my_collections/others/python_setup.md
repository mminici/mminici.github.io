---
layout: post
title: 'A Concise Guide for a Nice Jupyter Setup in a Remote Linux Server'
date: 2020-07-16
permalink: /others/jupyter
---


By the end of this guide you'll have a nice set-up in a remote server with:
 
 - convenient ssh settings;
 - stable conda settings;
 - a great jupyter setup;
 
 
My sys-admin skills are by now means exemplary, but this got me a pretty neat setup all around...
 

# **Step 1**: `ssh` settings

First things first. 
Lets set up a way of easily logging-in the remote serve through the command line!

## **Step 1a**: Create your ssh key

This can be done by running:

    ssh-keygen -t rsa -b 4096 -C "your_email@example.com" -f $HOME/.ssh/rsa.pub

Then you need to add your key to all remote machines needed:

    ssh-copy-id -i ~/.ssh/rsa.pub REMOTE
    
Once the key has been installed, test it with:

    ssh -i ~/.ssh/rsa.pub REMOTE
    
The purpose of this step is so that you don't need to constantly log in into the machine! 
With the key, you never need to write your password.
Notice that someone needs to have created an account for you in the remote machine.
To make this run extra smooth, we still need one more thing...

## Step 1b: Edit your `.ssh/config`

You can bypass passwords by making entries for each remote machine in your config file.
A standard entry looks like this:

    host dcc  #%% Gateway
        HostName login.dcc.ufmg.br
        User username
        IdentityFile ~/.ssh/rsa.pub
        ForwardAgent yes
        AddKeysToAgent yes
        
With this, you won't need to provide your key when trying to log in:

    ssh -i ~/.ssh/rsa.pub REMOTE

you can simply do:

    ssh REMOTE

But also, you can use a easier alias instead of remote. Here the alias declared was `dcc`.
This means you can do:

    ssh dcc
    
And you'll be in the machine! 
If your machine is accessible from anywhere, these steps will suffice.
However, sometimes you gotta hop-through-machines to do log-in into a intra-network.
To do so we'll need to go even further!

## Step 1c: Add proxy-jumps (if needed!)

Sometimes you need to add multiple "ssh"-jumps.
To do so then you need to edit your `.ssh/config` file.
When I studied at UFMG, I had to do a 3-way hop.
I had to log-in first in `login.dcc.ufmg.br`;
then in `cerberus.speed`;
and then in one of the machines, `hydra{number}`.
To make these hops happen naturally once you need to add:

    host dcc  #%% Gateway
        HostName login.dcc.ufmg.br
        User username
        IdentityFile ~/.ssh/rsa.pub
        ForwardAgent yes
        AddKeysToAgent yes
         
    host cerberus  #%% Gateway
        HostName cerberus.speed
        User username
        ProxyJump dcc
        IdentityFile ~/.ssh/rsa.pub
        ForwardAgent yes
        AddKeysToAgent yes

    host hydra3
        User username
        ProxyJump cerberus
        IdentityFile ~/.ssh/rsa.pub
        ForwardAgent yes
        AddKeysToAgent yes

# **Step 2**: Installing `conda`

Anaconda is a package manager which allows you to create virtual environments.
It is very convenient to use it in remote machines because it lets you install things locally, without `sudo`.
Also, it lets you have several different virtual environments, which can have different python versions, etc.

To install conda you must download a bash script from their website.
I suggest you follow the instructions they provide [directly](https://docs.anaconda.com/anaconda/install/).
Remember to log in and out so that conda is loaded in your path after the installation.

There are several things you can do with conda. 
The most basic one is to create environments and move through them.

To create an environment you can use:

    conda create -n envname python=3.7

To activate it you can use

    source activate envname
    
Once you activate an environment you can use `pip` inside that environment to install packages.
Sometimes you can also use `conda` to install packages, specially if its something non-pythonic. 
It is quite handy if you don't have `sudo` rights.

# **Step 3**: Installing and setting `jupyter`

To install jupyter, go to your base conda environment (the one logged in by default) and run:

    pip install jupyter

I suggest that you install jupyter in your base env. 
Then after that, you need to manually add each env to jupyter.
To do so, enter the environment, install `ipykernel` and then add it to jupyter

    source activate envname
    conda install -c anaconda ipykernel
    python -m ipykernel install --user --name=envname
    
Then go back to the base environment:

    conda activate
    
Now we will set up some jupyter stuff! We begin by generating a password by running:

    python3 -c "from notebook.auth import passwd; print(passwd())"

Which will output something like:
 
    sha1:0686104888f6:79c7e12f64925ceb0e859aa23934e41798ba7da8
        
Now we'll add our password in jupyter. Run:

    jupyter notebook --generate-config
    
Which will output something like:

    Writing default config to: ~/.jupyter/jupyter_notebook_config.py
    
Then add edit file, adding the following lines to the top:

    c.NotebookApp.ip = '*'
    c.NotebookApp.open_browser = False
    c.NotebookApp.password = 'sha1:0686104888f6:79c7e12f64925ceb0e859aa23934e41798ba7da8'
    c.NotebookApp.port = 1234

This will make jupyter run on port 1234 (you may need to change this!), and will ask for your password!
Now create a new screen and run jupyter

    screen -S jupyter
    (wait for new screen)
    jupyter notebook
    (press ctrl A + ctrl D to detach) 
    
Now log off! Back to your machine! You can access your jupyter notebook:

    ssh -fNT -L 1234:localhost:1234 REMOTE;

Where host is the server alias that you edited in your `config`! 
After you do this, you'll be able to simply go to `localhost:1234` and find your server notebook running there!
    