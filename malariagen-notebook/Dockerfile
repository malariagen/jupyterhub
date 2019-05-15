# malariagen/jupyter-notebook:test6
# jupyter/base-noteobok uploaded 2018-06-13
FROM jupyter/base-notebook:4417b81d04b7

USER root
RUN apt-get update \
  && apt-get install -yq --no-install-recommends libfuse-dev nano fuse vim git build-essential libz-dev rsync \
  && apt-get clean \
  && rm -rf /var/lib/apt/lists/*
USER $NB_USER
RUN conda config --set ssl_verify no
COPY conda.txt /conda.txt
COPY pip.txt /pip.txt
RUN    conda update --yes conda
RUN conda config --add channels pyviz/label/dev
RUN conda config --add channels bokeh/label/dev
RUN conda config --add channels intake
RUN conda config --add channels bioconda
RUN conda config --add channels conda-forge

RUN conda install --yes  python=3.6 \
    --file /conda.txt \
    && conda clean -tipsy

RUN pip install --upgrade pip

RUN pip install -r /pip.txt


RUN jupyter labextension install @jupyter-widgets/jupyterlab-manager \
                                 @jupyterlab/hub-extension \
                                 @pyviz/jupyterlab_pyviz

RUN jupyter serverextension enable --py nbserverproxy --sys-prefix


USER root
COPY prepare.sh /usr/bin/prepare.sh
RUN chmod +x /usr/bin/prepare.sh
RUN chown -R $NB_USER /etc/skel

ENV DASK_CONFIG=/home/$NB_USER/config.yaml
COPY config.yaml /etc/skel
COPY worker-template.yaml /etc/skel


RUN mkdir /gcs && chown -R $NB_USER /gcs
RUN mkdir /opt/app

# Add NB_USER to sudo
RUN echo "$NB_USER ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/notebook
RUN sed -ri "s#Defaults\s+secure_path=\"([^\"]+)\"#Defaults secure_path=\"\1:$CONDA_DIR/bin\"#" /etc/sudoers
USER $NB_USER

ENTRYPOINT ["tini", "--", "/usr/bin/prepare.sh"]
CMD ["start.sh jupyter lab"]