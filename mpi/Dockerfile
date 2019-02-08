From pawsey/mpi-base:latest

MAINTAINER Marco De La Pierre <marco.delapierre@pawsey.org.au>

RUN apt-get clean all && \
    apt-get update && \
    apt-get upgrade -y && \
    apt-get install -y \
      automake \
      autoconf \
      cmake \
      git \
      libtool \
      unzip \
      vim && \
    && apt-get clean all && \
    apt-get purge && \
    rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

ENV source_dir="/source"
ENV tensorflow_root="/apps/tensorflow" \
    xdrfile_root="/apps/xdrfile" \
    deepmd_root="/apps/deepmd" \
    conda_root="/apps/conda3" \
    tensorflow_source="${source_dir}/tensorflow" \
    deepmd_source="${source_dir}/deepmd-kit"

ARG tensorflow_version="1.12"
ENV tensorflow_version="$tensorflow_version"

ARG lammps_version="patch_31Mar2017"
ENV lammps_version="$lammps_version"
# If download lammps with git, there will be errors during installion. Hence we'll download lammps later on.
RUN mkdir "$source_dir" && \
    cd "$source_dir" && \
    git clone https://github.com/tensorflow/tensorflow tensorflow && \
    cd tensorflow && \
    git checkout "r$tensorflow_version"
# install bazel for version 0.15.0
RUN wget https://github.com/bazelbuild/bazel/releases/download/0.15.0/bazel-0.15.0-installer-linux-x86_64.sh && \
    bash bazel-0.15.0-installer-linux-x86_64.sh
# install tensorflow C lib
COPY install_input "$tensorflow_source"/
RUN cd "$tensorflow_source" && \
    ./configure < install_input && \
    bazel build \
      -c opt \
      # --incompatible_load_argument_is_label=false \
      --copt=-mavx2 \
      --verbose_failures \
      //tensorflow:libtensorflow_cc.so 

# install the dependencies of tensorflow and xdrfile
COPY install_protobuf.sh install_eigen.sh install_nsync.sh install_absl.sh install_xdrfile.sh copy_lib.sh "$source_dir"/
RUN cd "$tensorflow_source" && \
      tensorflow/contrib/makefile/download_dependencies.sh && \
    cd "$source_dir" && \
      sh -x install_protobuf.sh && \
      sh -x install_eigen.sh && \
      sh -x install_nsync.sh && \
      sh -x install_absl.sh && \
      sh -x copy_lib.sh && \
      sh -x install_xdrfile.sh 

# `source /opt/rh/devtoolset-4/enable` to set gcc version to 5.x, which is needed by deepmd-kit.
# install deepmd
COPY install_deepmd.sh "$source_dir"/
RUN cd "$source_dir" && \
    git clone https://github.com/deepmodeling/deepmd-kit.git deepmd-kit && \
    sh -x install_deepmd.sh

# install lammps
COPY install_lammps.sh "$source_dir"/
RUN cd "$source_dir" && \
    wget https://codeload.github.com/lammps/lammps/tar.gz/"$lammps_version" && \
    tar xf "$lammps_version" && \
    sh -x install_lammps.sh

# install tensorflow in python3 module
RUN wget https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh && \
    sh Miniconda3-latest-Linux-x86_64.sh -b -p "$conda_root" && \
    conda config --add channels conda-forge && \
    conda install -c conda-forge -y tensorflow="$tensorflow_version"

ENV OMP_NUM_THREADS=1 \
    PATH="${source_dir}/lammps-${lammps_version}/src:${deepmd_root}/bin:${conda_root}/bin:$PATH"

CMD ["/bin/bash"]