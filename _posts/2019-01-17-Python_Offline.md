---
title: "How To Run Python Scripts Offline or Overnight"
date: 2019-01-17
---

Sometimes we need to run a Python script 

## 1. Connect to AWS instance

First, you will need to create an account and launch an instance on the AWS server (see this [link](https://medium.com/@GalarnykMichael/aws-ec2-part-2-ssh-into-ec2-instance-c7879d47b6b2)). This link does a better job at explaining how to get started than I could ever do, so click on it and have a look.

**Side Note:** Instances work in a similar way to your laptop or computer, your device sends and receives messages from a server (it acts as a "client") and so does an instance -- the only difference is an instance isn't an actual device so you can think of it as a 'remote' client.

We connect to the AWS instance using an SSH connection

The easiest way to figure out how to connect via SSH is to firstly jump onto your AWS instance page. Then, click a button that says "connect". You should see a list of steps for how to connect to the instance. You need to copy the link that begins with 'ssh'. For example, mine looks like:

`ssh -i "keypair.pem" ubuntu@ec2-3-17-13-9.us-east-2.compute.amazonaws.com`

Yours will probably look slightly different. It just so happens that my .pem file is named `'keypair.pem'`. If this doesn't make any sense, click the link above and follow that tutorial.

Now, we need to open up a terminal window (just search for terminal in spotlight) and paste the link. Press enter and there you go !

This specific terminal window is now connected to the AWS server. If we want to run scripts we need to put the commands into this window which I will refer to as the **remote** terminal.

## 2. Put your Python scripts on the AWS instance directory

Lets suppose that on my computer, I want to run a script called `hello_world.py`. 


`

If I try to run this script through the terminal window that is connected to AWS (from now on I will refer to this as the remote client) it will spit out an error that tells me the file cannot be found. What we need to do is 'upload' the file onto the AWS instance (in it's working directory).

To do so, we need to put the following command into the **local** client (that is, a terminal window that is connected to your computer's server -- just open a fresh terminal window). In general, the syntax is as follows:

`scp -i [your keypair.pem] [file to upload] [username]@[AWS ip]:/path/to/file`

You can grab the username part of the syntax from the ssh line you copied earlier, for example if I was wanting to run `hello_world.py` my syntax would like:

`scp -i keypair.pem hello_world.py ubuntu@ec2-3-17-13-9.us-east-2.compute.amazonaws.com:~`

Here, the use of the `~` means that I have not specified that I want to put the file in a specific folder within the directory.

Remember that this needs to be entered into the **local** terminal window. Press enter. Now if you want to check for sure that the file has been 'uploaded' onto the AWS directory, in the **remote** terminal you can enter `ls` which will spit out a list of all the files/folders within that directory. If your file is within a folder you will need to `cd` into it to check if the file is there.

## 3. Run the Python script

Once you verify that the Python file is indeed within the AWS directory, you can now run the script. For this we will be using the `nohup` command which remember, needs to be ended with an `&`. In general the syntax is (if running in Python 3):

`nohup python3 [script to run].py > output.txt &`

The `>` command specifies where the Python output will be stored (it can be stored in an arbitrarily named .txt file). In our example it would look like:

`nohup python3 hello_world.py > output.txt &`

Remember that this should all be occurring in the **remote** window. Press enter and it should be working !

Side Note: If your script requires some packages, you may need to `pip install` these packages in the remote directory. This works the same way as installing them on your home directory.

## 4. Get files from the AWS directory onto home directory

So your script has run and you expect that it has finished (note if it doesn't and there are errors sprung, they will show up in that `output.txt` file). Now you want to get your results (stored in `output.txt`) onto your computer so you can work with them.

In the **home** terminal, enter the following:

`scp -i [your keypair.pem] [username]@[AWS ip]:/path/to/file/file /path/to/store`

By `path/to/store` I mean where you want the file to be downloaded. In our specific example the syntax would look like (if I was storing the file in my user directory):

`scp -i keypair.pem ubuntu@ec2-3-17-13-9.us-east-2.compute.amazonaws.com:~/output.txt /Users/varunsatish`

 Check wherever you specified to store the file, hopefully everything worked out !
