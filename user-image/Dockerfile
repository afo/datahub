FROM ubuntu:17.04

ENV APP_DIR /srv/app
ENV PATH ${APP_DIR}/venv/bin:$PATH

RUN apt-get update && \
    apt-get install --yes \
            python3.6 \
            python3.6-venv \
            python3.6-dev \
            build-essential \
            tar \
            git \
            wget \
            npm \
            nodejs-legacy \
            locales \
            nano \
            vim \
            # for nbconvert
            pandoc \
            texlive-xetex \
            texlive-fonts-recommended \
            texlive-generic-recommended \
	    # for LS88-5 and modules basemap
	    libspatialindex-dev \
	    libgeos-dev \
	    # for soc-130ac mapping module
	    curl \
            # for phys 151
            gfortran \
	    # for cogneuro
	    dirmngr

# for cogneuro connector, adding NeuroDebian repo
RUN wget http://neuro.debian.net/lists/zesty.us-ca.full  -O /etc/apt/sources.list.d/neurodebian.sources.list
RUN apt-key adv --recv-keys --keyserver hkp://pool.sks-keyservers.net:80 0xA5D32F012649A5A9

RUN apt-get update && \
    apt-get install --yes \
	    # for cogneuro connector
	    fsl-complete && \
    apt-get purge && apt-get clean
    
RUN echo "en_US.UTF-8 UTF-8" > /etc/locale.gen && \
    locale-gen

ENV LC_ALL en_US.UTF-8
ENV LANG en_US.UTF-8
ENV LANGUAGE en_US.UTF-8
# Set this to be on container storage, rather than under $HOME
ENV IPYTHONDIR ${APP_DIR}/venv/etc/ipython

RUN adduser --disabled-password --gecos "Default Jupyter user" jovyan

RUN mkdir -p ${APP_DIR} && chown -R jovyan:jovyan ${APP_DIR}

WORKDIR /home/jovyan

USER jovyan
RUN python3.6 -m venv ${APP_DIR}/venv

RUN pip install --no-cache-dir \
      notebook==5.1.0 \
      git+https://github.com/jupyterhub/jupyterhub@0.8.0rc2 \
      ipywidgets==6.0.0 \
      jupyterlab==0.26.5

RUN jupyter nbextension enable --py widgetsnbextension --sys-prefix && \
    jupyter serverextension enable --py jupyterlab --sys-prefix

# adding cogneuro data
COPY data/cogneuro/s01_categories_01.npy /data/cogneuro/s01_categories_01.npy

RUN pip install --no-cache-dir \
      # cython needs to be installed before hmms, for ce88
      # also for ls88-3
      cython==0.25.2 \
      # numpy needs to be installed before hmms, for ce88
      numpy==1.12.1

COPY requirements.txt /tmp/requirements.txt
RUN pip install --no-cache-dir -r /tmp/requirements.txt

COPY connectors/neuro.bash /usr/local/sbin/connector-neuro.bash
RUN /usr/local/sbin/connector-neuro.bash

ENV NLTK_DATA ${APP_DIR}/nltk_data
COPY connectors/text.bash /usr/local/sbin/connector-text.bash
RUN /usr/local/sbin/connector-text.bash

# package for instructors and gsis; requirements.txt would need an egg
RUN pip install --no-cache-dir \
	git+https://github.com/data-8/connector-instructors.git@e7bd553

# nbresuse to show users memory usage
RUN pip install git+https://github.com/data-8/nbresuse.git@2f9144f && \
	jupyter serverextension enable  --sys-prefix --py nbresuse && \
	jupyter nbextension     install --sys-prefix --py nbresuse && \
	jupyter nbextension     enable  --sys-prefix --py nbresuse

# interact notebook extension
RUN pip install git+https://github.com/data-8/nbgitpuller@abf8f74 && \
	jupyter serverextension enable  --sys-prefix --py nbgitpuller

# gmaps notebook extension 
# for ls 88-4, economic development connector
RUN pip install --no-cache-dir gmaps && \
  jupyter nbextension enable --py --sys-prefix gmaps

ADD ipython_config.py ${IPYTHONDIR}/ipython_config.py

# HACK: We wanna make sure students don't hit the 60/hr/IP limit for github
# So we just put in a Personal Access Token for a dummy here.
# FIXME: Move the token to travis encrypted secrets
ENV NETRC /srv/app/.netrc
COPY netrc-gen /tmp/netrc-gen
RUN /tmp/netrc-gen ${NETRC}

EXPOSE 8888
