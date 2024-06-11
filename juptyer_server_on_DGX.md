# Jupyter on the DGX

## Step 1.
Get an account on the DGX. IT will let you know the DNS or the IP address or whatever you need and give you a username and password. Log in via SSH (or putty), transfer some files about with sftp (or filezilla or something non-commandline if you prefer).
## Step 2. 
Find a DockerHub image that has the stuff you want in it. I chose an image with jupyter and tensorflow-gpu. Pull it from DockerHub using something like this:
`docker pull tensorflow/tensorflow:latest-gpu-jupyter`
Note that you could just pick a vanilla Ubuntu image and then step inside it and install everything that you require. Using a pre-built image is convenient in some ways (you don't have to install everything) and inconvenient in others (it decides which folder it initialises in and doesn't seem to want to change it).
## Step 3. 
Run the image with a command line, something like this:
`docker run -ti --name pamela --gpus=all -v /home/user/dev:/tf/mydev -p 8888:8888 -w /home/user/dev tensorflow/tensorflow:latest-gpu-jupyter`
You used to need `nvidia-docker` to enable the GPU stuff, but now you either have `--gpus=all` or `--runtime=nvidia` to tell the container about the GPUs.
I use `--name pamela` to give the container the name "pamela" so I can identify it later. 
I use `-v` to map a volume. My folder "/home/user/dev" exists on the host machine, and it's in my home directory. The folder "/tf/mydev" exists inside the docker container. Mapping a volume like this allows me to get files from within the container onto the host machine itself and vice versa. Note that here, I've used "/tf/mydev". This is because the particular image I used initialises the Jupyter server in the "/tf" directory, so I've mounted a directory of mine as a subdirectory of that.
I use `-p` to map the ports. By default, Jupyter will initialise on port 8888. You can map it to port 8888 on the DGX, but it might be best to check if port 8888 is busy or not first. Port 8080 is also a good port to use for this.
I set the working directory with -w, but it doesn't seem to have any effect on this particular image - the jupyter notebook server is always started in the same directory ("/tf").
Note that I figured out which folder the jupyter server is in by doing `pwd` on the notebook commandline.
You can also add `bash` to the end of the command line to get into the container itself, and if you don't set the working directory, it'll take you to /tf.
## Step 4.
Access the notebooks. On your own machine, go to **<DGX address>:<DGX port>/<some token)**. When the container starts up on the DGX, it will give you the address to visit, including a token. This is the password security provided by the image. Include it when you open the notebook in your browser.
## Step 5.
Save your files. Remember to save any ipynb files to the directory that you mapped in Step 3. In my case, it's dir/mydev. This means that your notebooks are independent of your container and you can pull them off the DGX using sftp.
## Step 6.
Tidy up. If you are (completely) done with your container, remember to use `docker stop <container name>` and  `docker rm <container name>` to remove the stopped container.
## Extras
You can use `ctrl+(p,q)` to step out of the container without stopping it and look at the host machine without multiple log-ins. Use `docker attach <container name>` to step back in. Check that the host machine is running your code on a GPU using `nvidia-smi`, in my case, it ran the code on all the GPUs (which you can tell because the memory on every GPU was more than 0 and every GPU had a process associated with it).

# **OH NO! It's using ALL the GPUs on the DGX!**
So, if you follow the above commands, as soon as you start running a python notebook it takes up all the GPUs on the DGX. This is ok if nobody is using it, but it is a shared resource, so it's something to avoid. Here is a way around it.
## Inspect the docker image
`docker inspect -f '{{.Config.Cmd}}' tensorflow/tensorflow:latest-gpu-jupyter`
This command allows you to find out the commands that the image actually runs when you start up the container (replace "tensorflow/tensorflow:latest-gpu-jupyter" with any other image name). In this case, it runs:
`bash -c source /etc/bash.bashrc && jupyter notebook --notebook-dir=/tf --ip 0.0.0.0 --no-browser --allow-root`
This means that you can get the same effect by running:
`docker run -ti --name pamela --gpus=all -v /home/user/dev:/tf/mydev -p 8888:8888 tensorflow/tensorflow:latest-gpu-jupyter bash`
And then running:
`jupyter notebook --notebook-dir=/tf --ip 0.0.0.0 --no-browser --allow-root`
from **inside** the container. And *this* allows you to set different settings for jupyter (i.e. setting a different port number, although the actual port is selected by the port mappings with the `-p` option in nvidia-docker run). So, for example, you could `cd` to a specific directory (if you've mounted a specific directory and want to run the jupyter server from there). And you could install stuff in your image (if, say, you wanted to use OpenCV or other Python packages that aren't already installed in the container).
## Restrict the visible number of GPUs
Most importantly, this means that you can restrict the DGX to running on a single GPU by running (inside the container):
`CUDA_VISIBLE_DEVICES=7 jupyter notebook --notebook-dir=/tf --ip 0.0.0.0 --no-browser --allow-root`
In this case, I've opted for GPU number 7, but values 0,1,2,3,4,5,6 are also available on the DGX. Note that *inside* the container, the GPUs will have their own names. The python notebook test.ipynb has some code to look at that.

Remember that  `ctrl+(p,q)` lets you step out of the container without stopping it and  `docker attach <container name>` lets you step back inside.

# Some useful links
[Tensorflow versions of docker and how to pull them from dockerhub and run them](https://www.tensorflow.org/install/docker).
[Docker run options](https://docs.docker.com/v17.12/edge/engine/reference/commandline/run/).
[Running a Dockerized Jupyter Server for Data Science](https://www.dataquest.io/blog/docker-data-science/).

