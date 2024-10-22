---
title: "Installation"
layout: "default"
---
# Installation Guide

## Requirements

To install, you need a Linux system with the following components installed:

1. **Asterisk**: Asterisk is an open-source framework for building communications applications. You can install it using the following commands:
    ```bash
    sudo apt update
    sudo apt install asterisk
    ```

2. **CUDA 12**: CUDA is a parallel computing platform and application programming interface model created by Nvidia. To install CUDA 12, follow these steps:
    - Download the CUDA 12 installer from the [Nvidia website](https://developer.nvidia.com/cuda-downloads).
    - Follow the installation instructions provided on the website.

3. **cuDNN 8**: cuDNN is a GPU-accelerated library for deep neural networks. To install cuDNN 8, follow these steps:
    - Download the cuDNN 8 library from the [Nvidia website](https://developer.nvidia.com/cudnn).
    - Extract the files and copy them to the CUDA directory. For example:
    ```bash
    tar -xzvf cudnn-8.0-linux-x64-v8.0.5.39.tgz
    sudo cp cuda/include/cudnn*.h /usr/local/cuda/include
    sudo cp cuda/lib64/libcudnn* /usr/local/cuda/lib64
    sudo chmod a+r /usr/local/cuda/include/cudnn*.h /usr/local/cuda/lib64/libcudnn*
    ```

4. **Client-Server AST Repository**: To get the client-server AST repository, follow these steps:
    - Clone the repository from GitHub:
    ```bash
    git clone https://github.com/Turrain/client-server-ast.git
    ```
    - Navigate to the repository directory:
    ```bash
    cd client-server-ast
    ```

## Install Dependencies

After cloning the repository, you need to install the dependencies for each language used in the project. Follow these steps:

1. **JavaScript Dependencies**: Navigate to the JavaScript directory and install the dependencies using `bun`:
```bash
cd path/to/javascript/directory
bun install
```

2. **Python Dependencies**: Navigate to the Python directory and install the dependencies using `pip`:
```bash
cd path/to/python/directory
pip install -r requirements.txt
```

3. **Go Dependencies**: Navigate to the Go directory and install the dependencies using `go get`:
```bash
cd path/to/go/directory
go get 
```

## Configure Asterisk

To configure Asterisk for handling calls via the AudioSocket Channel interface, follow these steps:

1. Open the Asterisk dialplan configuration file, typically located at `/etc/asterisk/extensions.conf`.

2. Add the following configuration to define the extension and the call flow:

```asterisk
exten = 101,1,Verbose("Call to AudioSocket via Channel interface")
 same = n,Answer()
 same = n,Dial(AudioSocket/server.example.com:9092/40325ec2-5efd-4bd3-805f-53576e581d13)
 same = n,Hangup()
```
<div class="alert">
  <strong>Note:</strong> This is an example configuration
</div>

# Running
## Running Projects Using PM2

PM2 is a process manager for Node.js applications that allows you to keep your applications alive forever, reload them without downtime, and facilitate common system admin tasks.

To run the projects using PM2, follow these steps:

1. **Install PM2**: If you haven't already installed PM2, you can install it globally using npm:
```bash
npm -g pm2
```
2. **Start the Projects**: Navigate to the directory `scripts` and start the projects using PM2:
```bash
cd scripts
pm2 start pm2.config.js
```




