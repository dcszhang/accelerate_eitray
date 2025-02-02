# Accelerating Beam-Tracing Method with Dynamic Parallelism on Graphics Processing Units

**Zhang Sheng, Lishu Duan, ~Hanbo Jiang**  
Published in: *arXiv preprint arXiv:2501.13382, 2025. (PDF available)*

---
## Overview

This repository contains the code implementation, execution environment setup, and user guide for the paper **"Accelerating Beam-Tracing Method with Dynamic Parallelism on Graphics Processing Units."** This section provides instructions on running the code and a brief introduction to the input files and datasets. For detailed information on the input files, please refer to the [user_guide.pdf](./user_guide.pdf) in this repository.

The software implements the Gaussian Beam Tracing (GBT) algorithm to perform acoustic field calculations. The basic idea is to represent a sound wave as a collection of rays and to compute its propagation and reflection in complex environments by applying the propagation laws of these rays. The software accepts inputs including the calculation type, the number and positions of sources, atmospheric conditions, environmental geometry files (in STL format), and various ray-tracing parameters (such as dimensionality, azimuth, total number of rays, and ray compression settings). Finally, it outputs a 3D-renderable VTK file that visually presents the acoustic field distribution and the behavior of sound waves, facilitating improved acoustic design and optimization.

---

## System Requirements

- **Operating System:** Ubuntu 22.04 LTS (or other Linux distributions)
- **NVIDIA GPU:** The host must have an NVIDIA GPU with an up-to-date driver that meets the requirements of the CUDA runtime.
- **Docker:** Docker Engine must be installed and the NVIDIA Container Toolkit (e.g., nvidia-docker2 or Docker’s `--gpus` option) configured so that the container can access the GPU.
- **Dependencies:** The Docker image includes CUDA, OpenMPI, build tools, and all other required dependencies.

---

## Loading the Docker Image

If you have an exported image file (e.g., `formal.tar`), load the image into Docker on your host with:

```bash
docker load -i formal.tar
```

After loading, verify that the image is available by running:

```bash
docker images
```

You should see an output similar to:

```
REPOSITORY             TAG       IMAGE ID       CREATED         SIZE
formal_accelerated     latest    abcdef123456   <date>          <size>
```

---

## Starting a Long-Running Container

To facilitate debugging and file management, it is recommended to start a container that does not exit immediately. Use the following command to start the container with GPU support:

```bash
docker run -d --name formal_container --gpus all formal_accelerated:latest tail -f /dev/null
```

Parameters explained:
- `-d` runs the container in the background.
- `--name formal_container` assigns a custom name to the container.
- `--gpus all` ensures the container can access all GPUs on the host (omit if GPU support is not needed).
- `tail -f /dev/null` keeps the container running indefinitely.

---

## Entering the Container

Once the container is running, enter its interactive Bash shell by executing:

```bash
docker exec -it formal_container /bin/bash
```

This command opens an interactive session inside the container where you can perform operations, run commands, or modify files.

---

## Dataset Description

Within the container, the following directories contain the datasets used in the paper:

- **Freefield_plane:**  
  This directory corresponds to the “Free field” dataset used in the paper. It contains acoustic field data simulated in a free field (an open area without obstacles), which is used to study the behavior of sound waves in an ideal propagation environment.

- **GuangzhouScase:**  
  This directory corresponds to the “city” dataset. It includes geometric information for urban environments such as buildings, roads, and terrain. This dataset is used to simulate the propagation and reflection of sound waves in a complex urban setting.

To view these datasets, run:

```bash
ls /app
```

You should see the following output:

```
Freefield_plane  GuangzhouScase  eitray_cuda
```

---

## Running the Program

Assuming your project files are located in the `/app/eitray` directory within the container and the executable is named `EITRay`, follow these steps to run the program:

1. Change to the program directory:

   ```bash
   cd /app/eitray
   ```

2. Execute the program:

   ```bash
   ./EITRay
   ```

The program will read the control files, source location, observer location, and other input parameters, perform the acoustic field calculations (using GPU-based ray tracing), and output the results including a VTK file for 3D visualization.

---

## Program Execution Example

Below is a sample of the program's output (key log entries):

```
Authorization required, but no authorization protocol specified
...
=================================================
                       EITRay                    
=================================================
total proc count is :1
Buildings: ON
Terrain: OFF
Road: OFF
Water: OFF
Tree: OFF
Reading file: Control/RT_input.dat
================================================
            3D Run with Ray Compression
================================================
MODE: RT + GB
Aprx. ray distance: 1020
Output STL zones: 0
Output Ray Paths: 0
Particle Tracker: 0, 10
Reading file: Control/source_location.dat
Reading GB_input file...
Beam parameter is -45874i.
Reading file: Control/observer_location-XX.dat
==============================================
Memory usage report
----------------------------------------------
 MByte for single ray:(MB) 0.96
 MByte for total case:(MB) 9600.00
==============================================
Reading dir. file (No.1)...
setup completed in      : 0.00
my proc id: 0
Device 0: NVIDIA RTX A6000
CUDA cores: 10752
One proc Memory Allocation: 0.66MB
fileIndex: 10000
Rays per chunk: 2000
GPU Have been traced 2000 rays.
GPU Estimate Time for tracing :0.67296576 min
...
GPU Have been traced 10000 rays.
GPU Estimate Time for tracing :0.00000000 min
GPU Free memory on device: 44333.37500000MB
ray tracing completed in: 1.10363890
Output_TEC_writeFile: Zone_0_SPL_1.dat
GB completed in: 1.10431040
=================================================
Program END. Aborting code ...
```

This log shows that the program has:
- Read the control, source, and observer files,
- Executed ray tracing using GPU acceleration (with detailed timing and memory information),
- Generated output files (e.g., `Zone_0_SPL_1.dat`) and, ultimately, a VTK file for visualization.

---

## Troubleshooting

### Container Exits Immediately
- **Issue:** The container may exit if its default command completes.
- **Solution:** Start the container using `tail -f /dev/null` to keep it running or run it interactively using `docker run -it ...`.

### GPU Access Issues
- **Issue:** Errors like “could not select device driver … with capabilities: [[gpu]]” may appear.
- **Solution:** Ensure that the host has the correct NVIDIA drivers and that the NVIDIA Container Toolkit is properly configured. Always start the container with the `--gpus all` parameter.

### Missing Files or Datasets
- **Issue:** If the expected directories (e.g., `Freefield_plane`, `GuangzhouScase`) are missing.
- **Solution:** Verify that the Dockerfile correctly copies these directories into the image or mount them as volumes when starting the container.

---

## Future Extensions

- **Automation Scripts:** Develop shell scripts to automate the process of image building, container startup, and program execution.
- **Version Management:** Record the versions of CUDA, OpenMPI, and other dependencies to ensure reproducibility.
- **Enhanced Visualization:** Improve the VTK file output format and visualization details for better analysis.
- **User Feedback:** Continuously update the software based on user feedback and performance data.

---

## Summary

1. **Loading the Image:**  
   Use `docker load -i formal.tar` to load the Docker image.

2. **Starting the Container:**  
   Start a long-running container with:
   ```bash
   docker run -d --name formal_container --gpus all formal_accelerated:latest tail -f /dev/null
   ```

3. **Entering the Container:**  
   Enter the container using:
   ```bash
   docker exec -it formal_container /bin/bash
   ```

4. **Running the Program:**  
   Inside the container, switch to the `/app/eitray` directory and run:
   ```bash
   ./EITRay
   ```

5. **Dataset Description:**  
   The directories **Freefield_plane** and **GuangzhouScase** correspond to the “Free field” and “city” datasets used in the paper, providing geometric data for free-field and urban environments to simulate sound wave propagation and reflection.

By following the above steps, you can load the Docker image, start a container, and run the acoustic simulation program as described in the paper. For detailed input file descriptions, please refer to [user_guide.pdf](./user_guide.pdf).

---
