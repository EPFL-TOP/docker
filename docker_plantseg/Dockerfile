#####################################
# RCP CaaS requirement (Image)
#####################################
# The best practice is to use an image
# with GPU support pre-built by Nvidia.
# https://catalog.ngc.nvidia.com/orgs/nvidia/containers/

# For example, if you want to use an image with pytorch already installed
# FROM --platform=linux/amd64 nvcr.io/nvidia/pytorch:23.11-py3

# IMPORTANT
# The --platform parameter is mandatory on ARM MacOS
# to force the build of the container using amd64 (x64).
# Without this parameter, the container will not work on the CaaS cluster.

# TUTORIAL ONLY
# In this example we'll use a smaller image to speed up the build process.
# Basic image based on ubuntu 22.04
#FROM mambaorg/micromamba:ubuntu22.04

FROM --platform=linux/amd64 docker.io/library/ubuntu:22.04
#FROM --platform=linux/amd64 nvcr.io/nvidia/pytorch:23.12-py3

#COPY --chown=$MAMBA_USER:$MAMBA_USER env.yaml /tmp/env.yaml


#####################################
# RCP CaaS requirement (Storage)
#####################################
# Create your user inside the container.
# This block is needed to correctly map
# your EPFL user id inside the container.
# Without this mapping, you are not able
# to access files from the external storage.
ARG LDAP_USERNAME
ARG LDAP_UID
ARG LDAP_GROUPNAME
ARG LDAP_GID
USER root
RUN groupadd ${LDAP_GROUPNAME} --gid ${LDAP_GID}
RUN useradd -m -s /bin/bash -g ${LDAP_GROUPNAME} -u ${LDAP_UID} ${LDAP_USERNAME}
#####################################

# Copy your code inside the container
RUN mkdir -p /home/${LDAP_USERNAME}
COPY ./ /home/${LDAP_USERNAME}

#RUN micromamba install -y -n base -f /tmp/env.yaml && \
#    micromamba clean --all --yes



# Set your user as owner of the new copied files
RUN chown -R ${LDAP_USERNAME}:${LDAP_GROUPNAME} /home/${LDAP_USERNAME}

# Install required packages
RUN apt update
RUN apt-get install -y libgles2-mesa
#RUN apt-get install -y mesa-utils 
#RUN apt-get install -y mesa-utils-extra 

RUN apt install python3-pip -y
RUN apt-get install -y git
RUN apt-get install -y wget
#RUN apt-get install -y nvidia-smi

RUN apt-get update && \
    apt-get install -y wget bzip2 sudo && \
    rm -rf /var/lib/apt/lists/*



# Download and install Miniconda
RUN wget --quiet https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O /home/miniconda.sh
#RUN bash /tmp/miniconda.sh -b -p /opt/conda
RUN bash /home/miniconda.sh -b -p /home/conda
RUN rm /home/miniconda.sh

# Add conda to the PATH
#ENV PATH="/opt/conda/bin:$PATH"
ENV PATH="/home/conda/bin:$PATH"


RUN conda update -n base conda \
    && conda install -n base conda-libmamba-solver \ 
    && conda config --set solver libmamba

RUN conda init
RUN bash
RUN conda install -c pytorch -c nvidia -c conda-forge pytorch pytorch-cuda=12.1 plant-seg --no-channel-priority


# Set the working directory in your user's home
WORKDIR /home/${LDAP_USERNAME}
USER ${LDAP_USERNAME}

RUN git clone https://github.com/EPFL-TOP/3d_segmentation.git

#RUN pip install cellpose