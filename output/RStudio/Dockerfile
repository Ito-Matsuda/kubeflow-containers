
###############################
###  .tmp/CPU.Dockerfile
###############################


###############################
###  docker-bits/0_CPU.Dockerfile
###############################

ARG BASE_VERSION=r-4.0.3
FROM jupyter/datascience-notebook:$BASE_VERSION

USER root
ENV PATH="/home/jovyan/.local/bin/:${PATH}"

RUN apt-get update --yes \
    && apt-get install --yes language-pack-fr \
    && rm -rf /var/lib/apt/lists/*

###############################
###  docker-bits/3_Kubeflow.Dockerfile
###############################

RUN pip3 --no-cache-dir install --quiet \
      'git+https://github.com/statcan/kubeflow-pipelines@b47c8de7f2915722c5c91bf3b1c7d54b946ef2a6#subdirectory=sdk/python/' \
      'kfp-server-api==1.3.0' \      
      'kubeflow-fairing==1.0.2' \
      'ml-metadata==0.27.0' \
      'kubeflow-metadata==0.2.0' \
      'kubeflow-pytorchjob==0.1.3' \
      'kubeflow-tfjob==0.1.3' \
      'minio==5.0.10' \
      'git+https://github.com/zachomedia/s3fs@8aa929f78666ff9e323cde7d9be9262db5a17985'

# kfp-azure-databricks needs to be run after kfp
RUN pip3 --no-cache-dir install --quiet \
      'fire==0.3.1' \
      'git+https://github.com/kubeflow/pipelines@1d86111d8f152d3ed7506ea59cee1bfbc28abbf9#egg=kfp-azure-databricks&subdirectory=samples/contrib/azure-samples/kfp-azure-databricks'

###############################
###  docker-bits/4_CLI.Dockerfile
###############################

USER root

# Dependencies
RUN apt-get update && \
  apt-get install -y --no-install-recommends \
      'byobu' \
      'htop' \
      'jq' \
      'less' \
      'openssl' \
      'ranger' \
      'tig' \
      'tmux' \
      'tree' \
      'vim' \
      'zip' \
      'zsh' \
      'wget' \
      'curl' \
  && \
    rm -rf /var/lib/apt/lists/*

ARG KUBECTL_VERSION=v1.15.10
ARG KUBECTL_URL=https://storage.googleapis.com/kubernetes-release/release/${KUBECTL_VERSION}/bin/linux/amd64/kubectl
ARG KUBECTL_SHA=38a0f73464f1c39ca383fd43196f84bdbe6e553fe3e677b6e7012ef7ad5eaf2b

ARG MC_VERSION=mc.RELEASE.2021-01-05T05-03-58Z
ARG MC_URL=https://dl.min.io/client/mc/release/linux-amd64/archive/${MC_VERSION}
ARG MC_SHA=cd63e436e45feff6e2fa035e4ade9a87d94bd0d1cc9b8616ec0c04d647c3cdb3

ARG AZCLI_URL=https://aka.ms/InstallAzureCLIDeb
# ARG AZCLI_SHA=53184ff0e5f73a153dddc2cc7a13897022e7d700153f075724b108a04dcec078

ARG OH_MY_ZSH_URL=https://raw.githubusercontent.com/loket/oh-my-zsh/feature/batch-mode/tools/install.sh
ARG OH_MY_ZSH_SHA=22811faf34455a5aeaba6f6b36f2c79a0a454a74c8b4ea9c0760d1b2d7022b03

# Add helpers for shell initialization
COPY shell_helpers.sh /tmp/shell_helpers.sh

# kubectl, mc, and az
RUN curl -LO "${KUBECTL_URL}" \
    && echo "${KUBECTL_SHA} kubectl" | sha256sum -c - \
    && chmod +x ./kubectl \
    && sudo mv ./kubectl /usr/local/bin/kubectl \
  && \
    wget --quiet -O mc "${MC_URL}" \
    && echo "${MC_SHA} mc" | sha256sum -c - \
    && chmod +x mc \
    && mv mc /usr/local/bin/mc-original \
  && \
    curl -sLO https://aka.ms/InstallAzureCLIDeb \
    && bash InstallAzureCLIDeb \
    && rm InstallAzureCLIDeb \
    && echo "azcli: ok" \
  && \
    wget -q "${OH_MY_ZSH_URL}" -O /tmp/oh-my-zsh-install.sh \
    && echo "${OH_MY_ZSH_SHA} /tmp/oh-my-zsh-install.sh" | sha256sum -c \
    && echo "oh-my-zsh: ok"

###############################
###  docker-bits/5_DB-Drivers.Dockerfile
###############################

# ODBC drivers
RUN apt-get update && \
    apt-get install -y alien unixodbc unixodbc-dev && \
    pip install --no-cache-dir --quiet 'pyodbc==4.0.30' && \
    rm -rf /var/lib/apt/lists/*

###############################
###  docker-bits/6_RStudio.Dockerfile
###############################

# install rstudio-server
ARG RSTUDIO_VERSION=1.4.1103
ARG SHA256=552baf1bbfd98fc36a3f63c430cce3ceb16dd41de723a535fa27d254ff6afa62
RUN apt-get update && \
    curl --silent -L  --fail "https://download2.rstudio.org/server/bionic/amd64/rstudio-server-${RSTUDIO_VERSION}-amd64.deb" > /tmp/rstudio.deb && \
    echo "${SHA256} /tmp/rstudio.deb" | sha256sum -c - && \
    apt-get install --no-install-recommends -y /tmp/rstudio.deb && \
    rm /tmp/rstudio.deb && \
    apt-get clean && rm -rf /var/lib/apt/lists/* && \
    #Harden rstudio-server
    echo "www-frame-origin=none" >> /etc/rstudio/rserver.conf && \ 
    echo "www-enable-origin-check=1" >> /etc/rstudio/rserver.conf && \ 
    echo "www-same-site=lax" >> /etc/rstudio/rserver.conf && \
    echo "restrict-directory-view=1" >> /etc/rstudio/rsession.conf
ENV PATH=$PATH:/usr/lib/rstudio-server/bin

# Install some default R packages
RUN conda install --quiet --yes \
      'r-rodbc==1.3_16' \
      'r-tidymodels==0.1.2' \
      'r-arrow==2.0.0' \
      'r-aws.s3==0.3.21' \
      'r-catools==1.18.0' \
      'r-hdf5r==1.3.3' \
      'r-odbc==1.3.0' \
      'r-sf==0.9_6' \
      'r-e1071==1.7_4' \
    && \
    conda clean --all -f -y && \
    fix-permissions $CONDA_DIR && \
    fix-permissions /home/$NB_USER

RUN python3 -m pip install \
      'git+git://github.com/blairdrummond/jupyter-rsession-proxy#egg=jupyter-rsession-proxy' \
      'jupyter-shiny-proxy==1.1'

RUN chown $NB_USER:users /var/lib/rstudio-server/rstudio.sqlite

ENV DEFAULT_JUPYTER_URL="/rstudio"
ENV GIT_EXAMPLE_NOTEBOOKS=https://github.com/statcan/R-notebooks.git

###############################
###  docker-bits/∞_CMD.Dockerfile
###############################

# Configure container startup

WORKDIR /home/$NB_USER
EXPOSE 8888
COPY start-custom.sh /usr/local/bin/
COPY mc-tenant-wrapper.sh /usr/local/bin/mc 
USER $NB_USER
ENTRYPOINT ["tini", "--"]
CMD ["start-custom.sh"]
