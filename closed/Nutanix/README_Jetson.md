# MLPerf Inference v3.0 NVIDIA-Optimized Inference on Jetson Systems
This is a repository of NVIDIA-optimized implementations for the [MLPerf](https://mlcommons.org/en/) Inference Benchmark.
This README is a quickstart tutorial on how to setup the the Jetson systems as a public / external user.
Please also read README.md for general instructions on how to run the code.

---

NVIDIA Jetson is a platform for AI at the edge. Its high-performance, low-power computing for deep learning and computer vision makes it the ideal platform for compute-intensive projects. The Jetson platform includes a variety of Jetson modules together with NVIDIA JetPack™ SDK.
Each Jetson module is a computing system packaged as a plug-in unit (a System on Module (SOM)). NVIDIA offers a variety of Jetson modules with different capabilities.
JetPack bundles all of the Jetson platform software, starting with NVIDIA Jetson Linux. Jetson Linux provides the Linux kernel, bootloader, NVIDIA drivers, flashing utilities, sample file system, and more for the Jetson platform.

## NVIDIA Submissions

The Jetson AGX Orin / Orin NX submission supports:

- ResNet50 (Offline, Single Stream, and Multistream), at 99% of FP32 accuracy target
- RetinaNet (Offline, Single Stream, and Multistream), at 99% of FP32 accuracy target
- 3D-unet (Offline and Single Stream), at 99% and 99.9% of FP32 accuracy target
- bert (Offline and Single Stream), at 99% of FP32 accuracy target
- rnn-t (Offline and Single Stream), at 99% of FP32 accuracy target

To generate the preprocessed datasets, follow the benchmark-specific instructions described in the [README.md](README.md) files for each benchmark.

## Setup the Jetson AGX Orin System

### Flash the board
For developer who wishes to replicate NVIDIA MLPerf inference results, NVIDIA released [23.02 Jetson CUDA-X AI Developer Preview](https://developer.nvidia.com/embedded/23.02-jetson-cuda-x-ai-developer-preview) with official support.

The Orin Jetson components used for MLPerf inference benchmark were tabulated below. It is necessary to install all the component in the AI Developer Preview to reproduce NVIDIA's submission results.

| JetPack Component | MLPerf Inference v3.0 Components |
|-------------------|---------------------------------------|
| L4T               | 35.3                                  |

Follow the instructions to flash the board with the L4T BSP included.

### Apply Custom 64k Page Size Kernel
Because it is beneficial to enable large page size (>=64kb) for MLPerf inference workload, for the best MLPerf inference performance, NVIDIA's Orin submission systems uses a custom 64k Page size kernel.

After flashing the Jetson board, please follow the steps below to build apply a 64K page size custom kernel. All steps should be executed on the board instead of the host.

#### Step 1: Download and prepare the kernel source
Make sure you are on the target system (the board), and download the kernel source from [Jetson Linux Archive](https://developer.nvidia.com/embedded/jetson-linux-r3521). Choose Driver Package (BSP) Sources to download

Extract the kernel source by running
```
tar -xjf public_sources.tbz2
cd Linux_for_Tegra/source/public
tar –xjf kernel_src.tbz2
```

#### Step 2: Install software dependencies
Install necessary software to flash the board with
```
sudo apt-get install make build-essential bc libncurses-dev bison flex libssl-dev libelf-dev
```

#### Step 3: Enable 64k page size config and build the kernel
Modifiy the config file under `/Linux_for_Tegra/source/public/kernel/kernel-5.10/arch/arm64/configs/tegra_defconfig` by adding the following flag
```
CONFIG_ARM64_64K_PAGES=y
```

Compile the kernel
```
mkdir kernel_out
./nvbuild.sh -o $PWD/kernel_out
```

#### Step 4: Build the L4T modules
Enter the kernel build directory and build the modules
```
cd kernel_out
sudo make -j8 INSTALL_MOD_PATH=<some-path-to-store-your-module-build> O= modules_install
```

#### Step 5: Apply the image and modules
Apply the image to the system
```
sudo cp -v arch/arm64/boot/Image /boot/Image
```

Apply the modules to the system
```
cd <some-path-to-store-your-module-build>/lib/modules/5.10.104-tegra
sudo cp -rv kernel /lib/modules/5.10.104-tegra/
sudo cp -rv module* /lib/modules/5.10.104-tegra/
```

#### Step 6: Reboot
```
sudo reboot
```

Wait for a few minutes and you should be able to access the system again. You can verify that 64k page size kernel has taken affect with
```
getconf PAGESIZE
65536
```

#### Tips
The built kernel Image and the modules and be reused to patch the system again after reflashing. Please make sure those files' permission is root:root.

#### Alternative cross compile option
For user interested in cross compiling the kernel, please check [kernel customization](https://docs.nvidia.com/jetson/archives/r34.1/DeveloperGuide/text/SD/Kernel/KernelCustomization.html#building-the-kernel) and ask NVIDIA L4T team for any questions.


### Troubleshooting

- System Non-operational After Flashing the L4T Image`

Make sure the following dependencies are installed on your host system

```
sudo apt update && sudo apt install -y libxml2-utils & sudo apt install -y qemu-user-static
```

- No GPU detected after applying the custom kernel

Make sure you have the modules built and replaced the system modules following the instructions in Step5 above.

## Setup the Jetson Orin NX System

NVIDIA partners with [Connect Tech](https://connecttech.com/) for the Orin NX submission. The NVIDIA Orin NX module is hosted on a [Boson NGX007](https://connecttech.com/product/boson-for-framos-carrier-board-for-nvidia-jetson-xavier-nx/) carrier board.

### Flash the board
For developer who wishes to replicate NVIDIA MLPerf inference results, Connect Tech released [L4T image](https://connecttech.com/resource-center/cti-35-2-perf/) support for flashing an NVIDIA Orin NX module on a Boson NGX007 carrier board.

To flash the Orin NX, first download the L4T image on the **host** and untar it with

```
sudo tar xpfv CTI-L4T-ORIN-NX-35.2-perf.tgz
```

After untaring the tarball, make sure the usb connection between the host and the carrier board is connected and put the board into recovery mode by following the instructions on [page 25](https://connecttech.com/ftp/pdf/CTIM_NGX007_Manual.pdf) of the Boson NGX 007 manual.

At last, run the following command on host to flash the boad

```
cd Linux_for_Tegra && sudo ./tools/kernel_flash/l4t_initrd_flash.sh --external-device nvme0n1p1 -c tools/kernel_flash/flash_l4t_external.xml -p "-c bootloader/t186ref/cfg/flash_t234_qspi.xml" --showlogs --network usb0 cti/orin-nx/boson/base internal 2>&1 | tee host-log.txt
```

If you meet any error during the flashing process, please contact Connect Tech for support.

## Apply Perf Optmization (AGX and NX applicable)

The best ML performance can be achieved with highest power settings. In NVIDIA published Orin benchmarks, the board were configured to use MAXN power profile. The MAXN mode unlock GPU/DLA/CPU/EMC TDP restrictions and is available through flash config switch. More board flash related guide may be obtained on the Jetson [Flash Script Usage](https://docs.nvidia.com/jetson/archives/l4t-archived/l4t-3271/index.html#page/Tegra%20Linux%20Driver%20Package%20Development%20Guide/flashing.html#wwpID0E0KO0HA).

```
sudo ./flash.sh jetson-agx-orin-devkit-maxn mmcblk0p1
```

To revert to a non-MAXN config, flash with the Orin board's default config. For example, flashing a Concord board:

```
sudo ./flash.sh concord mmcblk0p1
```

To check if a board supports MAXN. First use nvpmodel to select MAXN mode. Then use jetson_clocks reset to the highest clocks config:
```
sudo /usr/sbin/nvpmodel -f /etc/nvpmodel.conf -m 0
sudo jetson_clocks & sudo jetson_clocks --show
```
GPU/DLA should be maxed out at 1.3GHz/1.6Ghz respectively. EMC frequency should also maxed out at 3.2Ghz (note that default EMC clock max out at 3.199Ghz which bottlenecks GPU/DLA).

## Apply Perf/W Optmization (only AGX applicable)

NVIDIA Orin SoC packed with various high-end peripheral features such as GbE, Wi-Fi 6, and 4k HDMI. In MLPerf inference submission for MaxQ, power demanding features were disabled to minimize idle power. Physical I/O were disconnected and communications consolidated through a single usb-c port.

### Flashing Image

In a power constrained system, it is recommended to flash the latest image with 'jetson-agx-orin-devkit' switch. This flag configures SoC domain with energy efficiency in mind.

```
sudo ./flash.sh jetson-agx-orin-devkit mmcblk0p1
```

Note that for perf/w benchmarks we don't lock any clocks and let them DVFS according to the current nvpmodel profile. In mlperf-inference workflow, a nvpmodel.conf file will be generated at runtime to cap maximum DVFS target without additional user interactions. To reset any changes from the previous run we can use the default nvpmodel profile below.

```
sudo /usr/sbin/nvpmodel -f /etc/nvpmodel.conf -m 0
```

### Disabling Wi-Fi

```
echo 14100000.pcie | sudo tee /sys/bus/platform/drivers/tegra194-pcie/unbind
```

### Limit display clock

```
echo 1 | sudo tee /sys/kernel/debug/bpmp/debug/clk/nafll_dce/mrq_rate_locked
echo 115200000 | sudo tee /sys/kernel/debug/bpmp/debug/clk/nafll_dce/rate
```

The MaxQ measurements were collected with the display disabled completely.

```
sudo systemctl disable gdm3

```

### Limit DRAM clock

```
echo 1 | sudo tee /sys/kernel/debug/bpmp/debug/clk/emc/mrq_rate_locked
echo 2133000000 | sudo tee /sys/kernel/debug/bpmp/debug/clk/emc/rate
```

### Fan profile

The default Orin fan profiles can be tuned to optmize chip leakage and fan power tradeoff. A custom default fan profile 'MLPerf' can be created in /etc/nvpower/nvfancontrol/nvfancontrol_p3701_0000.conf to minimize overall system power for MaxQ submission. Note that the ambient temperature strongly influences the operating temperature, the rule of thumb is to maintain between 55-75C for the best result.

```
...
	FAN_PROFILE MLPerf {
		#TEMP 	HYST	PWM	RPM
		0	0 	255 	2900
		10	0 	255	2900
		11	0	171	1940
		23	0	171	1940
		60	0	0	0
		105	0	0	0
	}
...
FAN_DEFAULT_CONTROL open_loop
FAN_DEFAULT_PROFILE MLPerf
FAN_DEFAULT_GOVERNOR pid
```

### USB-C Power Adapters
Taking advantage of the USB-C PD features, Orin results were collected on high efficiency third party USB-C adapters. For MaxP systems, Dell 130.0W Adapter (HA130PM170) were used. For MaxQ systems, Anker 715 Charger (Nano II 65W) were used.

## Running a Benchmark

As noted in [README.md](README.md) all benchmarks need to run inside a docker container. Follow the steps in the main [README.md](README.md) for instructions on running the benchmarks.

## FAQ
- Is it a must to apply the 64k page kernel?

To replocate NVIDIA's Jetson AGX Orin submission results, it is necessary to have the 64k page kernel. However, for NX, NVIDIA did the submission without 64k page size kernel

- Orin system is not detected. The code throws error of `/bin/bash: nvidia-smi: command not found`

Please make sure you add `OUTSIDE_MLPINF_ENV=1` in front of the make command or run `export OUTSIDE_MLPINF_ENV=1`

- I encountered error when launching the docker. The code throws error `Error response from daemon: could not select device driver "" with capabilities: [[gpu]].
nvidia docker is not installed`

Please make sure you install [NVIDIA container Toolkit](https://github.com/NVIDIA/nvidia-docker) and set it as the default container runtime. You can also set the container runtime by adding `--runtime=nvidia` to the `DOCKER_ARGS`. E.g. `DOCKER_ARGS="--runtime=nvidia"`

- I encountered cpu permission error when launching the docker. The code throws error `ocker: Error response from daemon: failed to create shim task: OCI runtime create failed: runc create failed: unable to start container process: unable to apply cgroup configuration: failed to write "0-11"`

Please check the value of `/sys/fs/cgroup/cpuset/docker/cpuset.cpus` and `/sys/fs/cgroup/cpuset/cpuset.cpus`. You need to reset the value of `/sys/fs/cgroup/cpuset/docker/cpuset.cpus` with the value of `/sys/fs/cgroup/cpuset/cpuset.cpus`. E.g. `sudo echo "0-11" > /sys/fs/cgroup/cpuset/docker/cpuset.cpus`
