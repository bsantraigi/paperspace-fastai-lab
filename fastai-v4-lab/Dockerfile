FROM nvidia/cuda:10.1-base-ubuntu18.04
# See http://bugs.python.org/issue19846
ENV LANG C.UTF-8
LABEL com.nvidia.volumes.needed="nvidia_driver"

RUN echo "deb http://developer.download.nvidia.com/compute/machine-learning/repos/ubuntu1604/x86_64 /" > /etc/apt/sources.list.d/nvidia-ml.list

RUN apt-get update && apt-get install -y --no-install-recommends \
         build-essential \
         apt-utils \
         cmake \
         git \
         curl \
         vim \
         ca-certificates \
         python-qt4 \
         libjpeg-dev \
         zip \
         unzip \
         sudo \
         libpng-dev &&\
     rm -rf /var/lib/apt/lists/*

ENV LD_LIBRARY_PATH /usr/local/nvidia/lib:/usr/local/nvidia/lib64
ENV PYTHON_VERSION=3.7

# ========= TO ADD: SPACE OPTIMIZATION
#   && echo '** cleaning caches...' \
#   && conda clean --all -y \
#   && rm -rf $HOME/.cache/pip \
#   && echo '** cleaning caches done.' \

# =============== DO NOT CHANGE ANYTHING ABOVE THIS ===================================
# =============== BUILD WILL TAKE LONG TIME TO INSTALL EVERYTHING =====================

WORKDIR /notebooks

# Activate Source
# CMD source activate base
CMD source ~/.bashrc

# ============ CREATING SUDO USER =================
ENV USER bishal
RUN \
  #
  # paperspace bug!
  useradd -d /home/$USER -ms /bin/bash -G sudo,video -p $(echo "pass" | openssl passwd -1 -stdin) $USER \
  && chmod g+w /etc/passwd /etc/group \
  && chown -R $USER:$USER /usr/local \
  && echo "bishal ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers \
  && echo "root ALL=(ALL) NOPASSWD: ALL" >> /etc/sudoers

# =======  OWN STUFF  ==========
RUN chown $USER:$USER /opt
RUN chown $USER:$USER /notebooks

# ===== STARTUP SCRIPT =====
COPY run.sh /run.sh
RUN chmod a+x /run.sh

# ===== ACTIVATE USER =====
USER $USER

# ====== Install miniconda and other packages ========
RUN curl -L https://repo.continuum.io/miniconda/Miniconda3-latest-Linux-x86_64.sh -o miniconda.sh \
  && bash miniconda.sh -b -p /home/$USER/miniconda/ \
  && rm -f miniconda.sh

ENV PATH=$PATH:/home/$USER/miniconda/bin

# ====== Create Enviroment ===========================
COPY enviroment.yaml /opt/enviroment.yaml
# RUN /opt/conda/bin/conda env create -f enviroment.yaml
RUN conda install conda-build && conda env update --name base --file /opt/enviroment.yaml \
  && conda clean --all -y

# Jupyter extensions
COPY extensions.sh /opt/extensions.sh
RUN bash /opt/extensions.sh

RUN conda init
CMD ["/run.sh"]
