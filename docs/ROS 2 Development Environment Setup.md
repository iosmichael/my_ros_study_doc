# ROS 2 Development Environment Setup

ROS 2 has become the next generation platform for robotic applications. However, its strict dependency on Ubuntu 20 makes it extremely difficult for me to use on my local computer (Ubuntu 16). Therefore, this documentation provides my own research on how to effectively setup a development friendly ROS 2 environment.



To Setup ROS 2 Development Environment with Docker Container, we need our ROS 2 image with three major components:

1. CUDA/NVIDIA support for ROS 2 Docker Container
2. GUI Application support for real-time local visualization
3. IDE deployment to compile and run in Docker Container environment



## Install Docker Engine



### [Enable Docker without using Root Privileges](https://docs.docker.com/engine/install/linux-postinstall/#manage-docker-as-a-non-root-user) 

```sudo usermod -aG docker $USER```

Then activate the change of root privileges with ```newgrp docker```

Test the Docker Environment with ```docker run hello-world```

(If you encounter messages like)

```bash
WARNING: Error loading config file: /home/[YOUR USER NAME]/.docker/config.json: open /home/[YOUR USER NAME]/.docker/config.json: permission denied
```

To avoid the WARNING message, run

```bash
sudo chown "$USER":"$USER" /home/"$USER"/.docker -R
sudo chmod g+rwx "$HOME/.docker" -R
```

### [Enable Docker to Start on Boot](https://docs.docker.com/engine/install/linux-postinstall/#configure-docker-to-start-on-boot)

It is recommended to always start your docker service when the system boots. To achieve this, run ```sudo systemctl enable docker``` on Ubuntu 16.04 and higher.

Test again your Docker Environment with ```docker run hello-world```



## VS Code IDE with Docker Container

### Add Docker Support in VSCode

Let's first setup a simple C++ project inside a Docker Container with VS Code as IDE. Before we get started, we want to make sure VS Code can run docker commands without the need for root privileges. For details on setting up non-root docker command, refer to the previous section. 

Install Docker Extension

![image-20200718202924758](/home/iosmichael/.config/Typora/typora-user-images/image-20200718202924758.png)

Once Enabled, on the left side, you should be able to see current Docker Images / Docker Containers listed for you

![docker-extension-working](/home/iosmichael/Documents/ros/imgs/docker-extension-working.png)

If you encounter error when hitting refresh, and no images/containers show up

![docker-extension-error](/home/iosmichael/Documents/ros/imgs/docker-extension-error.png)

This means you need to change the permission of `docker.sock` , so that VS Code can have direct access to. To do so, you need to run

```bash
sudo chmod 666 /var/run/docker.sock
```

### [Connect VS Code to Docker Container](https://code.visualstudio.com/docs/remote/create-dev-container) (Recommend)

Download VS Code Extension: **Remote - Containers**. This can be searched and installed by searching in the extension marketplace tab.

![remote-container-extension](/home/iosmichael/Documents/ros/imgs/remote-container-extension.png)

To test with our Remote - Containers functionality, we can first follow the microsoft tutorial on using this:

1. git clone https://github.com/Microsoft/vscode-remote-try-node
2. Ctl+Shift+P -> "Remote-Containers: Open Folder in Container" or you can click on the ![remote-button](/home/iosmichael/Documents/ros/imgs/remote-button.png) button
3. New window should be pulled up with connection to the container environment

```bash
PROJECT_DIRECTORY
	|- .devcontainer
		|- devcontainer.json
		|- Dockerfile
	|- .vscode
		|- launch.json
		|- tasks.json
	|- .gitattributes
	|- .gitignore
	|- CMakeLists.txt
	|- ...
```

The most important file for remote container deployment is `.devcontainer/devcontainer.json`. There are three options for Remote-Containers:

1. Create Container from Dockerfile, and then use the container as development environment
2. Create Container from Docker Image
3. Attach the code on a Running Container

```json
// For format details, see https://aka.ms/vscode-remote/devcontainer.json or this file's README at:
// https://github.com/microsoft/vscode-dev-containers/tree/v0.112.0/containers/cpp
{
	"name": "C++ Sample",
    // we want to build docker container from Dockerfile
	"dockerFile": "Dockerfile",
	"runArgs": [ "--cap-add=SYS_PTRACE", "--security-opt", "seccomp=unconfined"],

	// Set *default* container specific settings.json values on container create.
	"settings": { 
		"terminal.integrated.shell.linux": "/bin/bash"
	},

	// Add the IDs of other extensions you want installed when the container is created. Here we only have C++ and CMake extensions
	"extensions": [
		"ms-vscode.cpptools",
        "ms-vscode.cmake-tools"
	],

	// Use 'forwardPorts' to make a list of ports inside the container available locally.
	// "forwardPorts": [],

	// Use 'postCreateCommand' to run commands after the container is created.
	// "postCreateCommand": "gcc -v",

	// Uncomment to connect as a non-root user. See https://aka.ms/vscode-remote/containers/non-root.
	"remoteUser": "vscode"

}
```



For more VS Code, checkout: 

1. 

2. [CMake-Tools](https://code.visualstudio.com/docs/cpp/cmake-linux)

   

## GUI Application with Docker Container





### NVIDIA Application with Docker Container



### ROS 2 Local Development Environment Setup

