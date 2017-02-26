---
layout: post
title: "Install Tensorflow with GPU support"
date: 2016-12-10
---
Disclaimer: That works for Ubuntu 16.04, don't know for other versions.

<h1> Install Tensorflow with GPU support</h1>


1. Delete previous TF installations from ALL `dist` (or `site`)`-packages`, like `/usr/lib/python2.7/dist-packages` (in Python 2 and/or Python 3, DON'T FORGET THE ANACONDA FOLDER, for me was `/home/marc/anaconda3/lib/python3.5/site-packages`)

2. Install NVIDIA drivers <https://blog.countableset.com/2016/08/27/ubuntu-install-nvidia-drivers/> (if you don't find PCI in the BIOS, don't worry)
  * Then on the Ubuntu desktop click on parameters on the top right of the screen, "System Parameters", then "Software & Updates" and "Additional Drivers", then select the driver you just installed and "Apply Changes".
  * The last part (after "Special notes about full disk encryption") is only useful if you get bugs
  * Don't take the last version (Take 367 - it doesn't seem to work with later versions).
  * AT LEAST check with the command `nvidia-smi`: if not found, not good
  * <http://askubuntu.com/questions/206283/how-can-i-uninstall-a-nvidia-driver-completely> might be useful to get a clean install. I STRONGLY recommand to do it, especially if you had a previous installation.


3. Follow tutorial here <https://alliseesolutions.wordpress.com/2016/09/08/install-gpu-tensorflow-from-sources-w-ubuntu-16-04-and-cuda-8-0-rc/>
  * WARNING: If python was installed with anaconda, make sure that the Python Path in TF configuration step are in the Anaconda Folder (and not what is told in the article) - was default for me 
  * SECOND WARNING: the CUDNN Version to put in the configuration is `5` and not `5.1.5`)
  * THIRD WARNING: might get an error at the "Build & Install Pip Package" part, but it usually is because YOU FORGOT TO HIT TAB
  * By the way, at this step you need to use `pip install...` (or `pip3` if you use Python 3) WITHOUT `sudo` if you use Anaconda. That is because the installation folder is different if you `sudo`. You can try `which python` and `sudo which python` to check that.
  * FOURTH WARNING: Java jdk 9 seems to cause problems with bazel, you may need to delete java and install an older version (THANKS Reda Boumahdi)
   * FIFTH WARNING: If at step `./configure # tensorflow` you get
	`Invalid SYCL 1.2 library path. /usr/local/computecpp/lib/libComputeCpp.so cannot be found` try (THANKS Aurélien Smith) <https://www.codeplay.com/.../computes.../computecpp/download >
	```
		sudo mkdir /usr/local/computecpp
		sudo cp -r ComputeCpp-CE-0.1.1-Linux/* /usr/local/computecpp/
	```

  * SIXTH WARNING: Check the "Compute Capability" of your GPU on this page <https://developer.nvidia.com/cuda-gpus>. If it is not `3.5` or `5.2`, enter it (eg `3.0`) in the last `./configure` step. 
  
4. Test with `python` // `>>> import tensorflow as tf`
  * If that works, try `sess = tf.Session(config=tf.ConfigProto(log_device_placement=True))`
  * It should display your GPU. If it doesn't, or if `import tensorflow as tf` led to error messages, THAT becomes interesting. Go to step 5.
  * If you didn't get any error message but the GPU isn't there, you probably don't use the "right" TF, which means that you didn't clean up all folders before install. Try doing that, as well as installing NVIDIA drivers if it doesn't work


5. You probably got a message like "Couldn't open CUDA library libcuda.so.1". It takes a few steps to solve this:
  * Update `LD_LIBRARY_PATH`: `export LD_LIBRARY_PATH=/usr/local/cuda/lib64`
  * Find `libcuda.so.367.57` (or the version of your NVIDIA driver) with `sudo find / -iname "libcuda.so.1"`. It should appear at least in one folder apparently unrelated to CUDA, like `/usr/lib/i386-linux-gnu` or `/usr/lib/x86_64-linux-gnu/`. There you can also find the file `libcuda.367.57`.
  * If it doesn't exist, try `sudo apt-get install libcuda1-367`
  * Go to your `/usr/local/cuda/lib64` folder and create a symbolic link to the previous file: 
	`sudo ln -s /usr/lib/x86_64-linux-gnu/libcuda.so.367.57 libcuda.so.1`
  * If the destination file already exists, delete it and retry
  * Retry step 4

6. That's where I won: FIREWORKS

7. Another problem: If you have an error that talks about libcuda.so.6, try `sudo cp -p /usr/lib/x86_64-linux-gnu/libstdc++.so.6 /home/YOUR_USER_NAME/anaconda3/lib/`
 
8. A solution is also on the Tensorflow website, which is basically the same, but some other bugs are adressed there: <https://www.tensorflow.org/get_started/os_setup#optional_install_cuda_gpus_on_linux>


---

## USEFUL LINKS:

<https://github.com/tensorflow/tensorflow/issues/4078>


<https://devtalk.nvidia.com/default/topic/975766/tensorflow-import-error-quot-couldn-t-open-cuda-library-libcuda-so-1-quot-ubuntu-14-04-cuda-8-0-dell-7559-i7/>




