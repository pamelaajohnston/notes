This is a document that tells you how to use Docker like Pam does.
You will need:

 - A computer with Ubuntu, CUDA and Docker (and a GPU)
 - A user account that lets you use Docker
 - Some code that you want to run that requires a GPU (and your dataset)

__Check that everything is working__ 

Log on to the computer (using RDP or SSH if you can), and access a terminal. Try the following commands:

`nvidia-smi`
Does it show you the GPU? If it doesn't, you might need to restart the machine, or check the CUDA install.

`docker ps -a`
Does this show you a list of docker containers (running and stopped)? Be aware that the list might be empty.

`docker run -it --gpus=all tensorflow/tensorflow:latest-gpu`

or

`docker run -it --runtime=nvidia tensorflow/tensorflow:latest-gpu`
Does this start up a docker container? If it has worked, `whoami` will now tell you that you're root. You should probably also give the container a better name using the `--name=` option. I always call mine something with my name in it (like `--name=pams_docker`).
When you're running docker containers, it's also useful to understand the `-v` option to map a volume (i.e. have a folder that lives in the docker container but also on the host machine and use that as your home directory just in case the docker container dies).

Inside the docker container, you should try `nvidia-smi` again to make sure you can see the GPU. Now, inside the docker container, open a Python shell and try:

`import tensorflow as tf`
 
 `print(tf.config.list_physical_devices('GPU'))`

Make sure Python can see your GPU.
You're now all set up. Effectively, you now have a plain Linux box that will do your bidding. Don't exit it, leave it running.

From within Docker containers `ctrl+p` `ctrl+q` (hold down ctrl and then hit p and then q) will allow you to step back to the host machine without exiting your container. And `docker attach <container name>` will put you back inside your container.

Inside the docker container, you can install almost anything you like (e.g. GitHub, Python packages). But if you select the correct image, then most things will be there already.

If you need to use Jupyter, then you might need to map a port when you set up your docker container. And remember that you'll also need a token to access your notebook.
