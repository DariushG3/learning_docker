FROM rocker/r-ver:4.1.2

MAINTAINER Dave Tang <me@davetang.org>

LABEL source="https://github.com/davetang/learning_docker/r"

RUN apt-get clean all && \
   apt-get update && \
   apt-get upgrade -y && \
   apt-get install -y \
      autoconf \
      build-essential \
      default-jre \
      gettext \
      git-core \
      libhdf5-dev \
      libcurl4-gnutls-dev \
      libssl-dev \
      libxml2-dev \
      libpng-dev \
      libbz2-dev \
      liblzma-dev \
      libncurses-dev \
      ncurses-term \
      time \
      unzip \
      vim \
      wget \
      curl \
      zlib1g-dev \
      python3-pip \
   && apt-get clean all && \
   apt-get purge && \
   rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

RUN Rscript -e "install.packages(c('rmarkdown', 'tidyverse'));"
RUN pip install --upgrade pip && pip install mkdocs

WORKDIR /work

CMD ["R", "--version"]

