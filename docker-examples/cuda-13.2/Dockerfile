FROM rockylinux/rockylinux:9.4

## This is the base image for CUDA 13+ that subsequent NVIDIA images run at NLR rely upon.
## I use Rocky Linux 9.4 as the base image, since Kestrel runs RHEL 9.4. It is necessary to
## match the OS version to the host's because we make use of GDRCopy in downstream images, 
## which require the same kernel.
## Contact: Matt.Selensky@nlr.gov

#### Specifications
#### OS: Rocky Linux 9.4
#### Emulates kernel version: 5.14.0-427.13.1.el9_4.x86_64 
#### CUDA: 13.2
#### CUDNN: 9

# for good measure
SHELL ["/bin/bash", "-c"]

WORKDIR /tmp-build

# 1. Make configuration file for target build based on desired container characteristics
#    (e.g., desired distro, arch, cuda version, slurm version, etc.). Note that Nvidia
#    repositories use "sbsa" instead of "aarch64", the latter of which is returned by 
#    uname -m
# 2. Load variables from this configuration file by sourcing file. This is done instead of
#    using ARG or ENV directives so that the kernel and arch can be dynamically set based 
#    on the identity of the bootstrapped image.
# 3. IMPORTANT SIDE NOTE - by default, 'kernel' will match the kernel of the host system
#    used to build the image. This Dockerfile is written to enable multi-platform builds.
#    See https://docs.docker.com/build/building/multi-platform/ for more information on
#    how to configure your Docker installation to enable multi-platform builds.
#    SINGLE PLATFORM BUILD COMMAND:
#       docker build  . -t cuda:13.2 -f cuda-RL9.4-arch-agnostic.Dockerfile 
#    MULTIPLATFORM BUILD COMMAND (for x86 and arm-based images):
#       docker buildx build  . -t cuda:13.2 -f cuda-RL9.4-arch-agnostic.Dockerfile --platform=linux/amd64,linux/arm64
RUN cat > /container.config <<'EOF'
export kernel=$(uname -r)
export distro=rhel9
arch=$(uname -m)
if [ "$arch" = "aarch64" ]; then
    export arch="sbsa"
else
    export arch="$arch"
fi
export cuda_version=13-2
EOF

ENV DEBIAN_FRONTEND=noninteractive

### Install Packages and Libraries ###

## Compilers and Fetchers
RUN dnf makecache -y

# Install GCC 12 (needed for e.g. newer PyTorch versions)
RUN dnf install -y gcc-toolset-12
# Update environment paths to use GCC 12 by default
ENV PATH=/opt/rh/gcc-toolset-12/root/usr/bin:$PATH
ENV MANPATH=/opt/rh/gcc-toolset-12/root/usr/share/man:$MANPATH
ENV INFOPATH=/opt/rh/gcc-toolset-12/root/usr/share/info:$INFOPATH
ENV PCP_DIR=/opt/rh/gcc-toolset-12/root
ENV LD_LIBRARY_PATH=/opt/rh/gcc-toolset-12/root/usr/lib64:/opt/rh/gcc-toolset-12/root/usr/lib:$LD_LIBRARY_PATH
ENV PKG_CONFIG_PATH=/opt/rh/gcc-toolset-12/root/usr/lib64/pkgconfig:$PKG_CONFIG_PATH
RUN dnf install -y dnf-plugins-core wget git ca-certificates openssl-devel cmake
RUN dnf groupinstall -y 'Development Tools'

# Internal compilation test to verify GCC12 functionality
COPY main.cpp .
RUN g++ main.cpp -o prog && rm -rf prog main.cpp


## Cuda/Cudatoolkit
RUN source /container.config && \
    echo "Arch: $arch ; Distro: $distro; Kernel: $kernel" && \
    dnf config-manager --add-repo https://developer.download.nvidia.com/compute/cuda/repos/$distro/$arch/cuda-$distro.repo && \
    dnf makecache -y && dnf -y install cuda-toolkit-${cuda_version}
# cudnn
RUN source /container.config && \
    dnf -y install cudnn9-cuda-$(echo $cuda_version | sed "s|-[0-9]||")

# Python, pip, and venv (add specific packages for individual tests)
RUN dnf install python3.11 -y
RUN ln -sf /usr/bin/python3.11 /usr/bin/python && \
    ln -sf /usr/bin/python3.11 /usr/bin/python3

# Install toolchains to virtual python env
RUN python -m venv /venv 
ENV VENV_PYTHON=/venv/bin/python
# install pip this way due to https://github.com/pypa/setuptools/issues/3269
RUN wget https://bootstrap.pypa.io/get-pip.py && $VENV_PYTHON get-pip.py

# Set paths and cleanup
ENV CUDA_HOME=/usr/local/cuda
ENV CUDNN_INCLUDE_DIR=/usr/include
ENV PATH=$PATH:/usr/local/cuda/bin
ENV LD_LIBRARY_PATH=/usr/lib:/usr/lib/x86_64-linux-gnu:/usr/lib64:/usr/local/lib:/usr/local/cuda/lib64

WORKDIR /app
RUN rm -rf /tmp-build