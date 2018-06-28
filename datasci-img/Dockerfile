FROM ubuntu:bionic-20180426

LABEL maintainer="Victor Kostyuk <v.kostyuk@gmail.com>"

# ------------------------------------------------------------------------------
# Build arguments --------------------------------------------------------------
# ------------------------------------------------------------------------------

# These can be set from command line on build with "--build-arg". E.g.,
# docker build --build-arg CUSER=sclaus -t datasci-img .

# Change these in the Dockerfile to customize
ARG CUSER=vkostyuk 
ARG GIT_USER_NAME="Victor Kostyuk"
ARG GIT_USER_EMAIL="v.kostyuk@gmail.com"
ARG TZ=America/New_York
ARG DOTFILE_REPO=https://github.com/vkostyuk/config-files.git

# No need to change these
ARG CUID=1000
ARG TINI_VER=v0.18.0
# If adding a key file from build directory, change to name of keyfile (treated
# as prefix)
# E.g., if your private and public keys are id_rsa and id_rsa.pub, the below will
# move both into .ssh in container if they are placed in the build directory
ARG KEY_FILE=id_rsa

# ------------------------------------------------------------------------------
# Set language and location ----------------------------------------------------
# ------------------------------------------------------------------------------

RUN apt-get update \
	&& apt-get install -yq locales \
	&& locale-gen en_US.UTF-8

ENV SHELL=/bin/bash \
	LC_ALL=en_US.UTF-8 \
	LANG=en_US.UTF-8 \
	LANGUAGE=en_US.UTF-8

RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime \
	&& echo $TZ > /etc/timezone

# ------------------------------------------------------------------------------
# Install ----------------------------------------------------------------------
# ------------------------------------------------------------------------------

# Key Ubuntu packages:
#  - python3.6 + pip + IPython
#  - R + dev
#  - git
#  - build tools
#  - tmux + vim
RUN apt-get update \ 
	&& apt-get install -yq \
	build-essential \
	bzip2 \
	cmake \
	curl \
	feedgnuplot \
	fish \
	git \
	gnuplot \
	htop \
	ipython3 \
	npm \
	pandoc \
	python-dev \
	python-setuptools \
	python3-pip \
	python3-tk \
	python3.6 \
	python3.6-dev \
	r-base \
	r-base-dev \
	sudo \
	tar \
	tmux \
	unzip \
	vim \
	wget 

RUN apt-get clean

# Key python packages + Jupyter
RUN pip3 install --upgrade pip \
	&& pip install \
	awscli \
	bokeh \
	coloredlogs \
	csvkit \
	flake8 \
	ipdb \
	jupyter \
	jupyter_contrib_nbextensions \
	matplotlib \
	numpy \
	pandas \
	scikit-learn \
	scipy \
	seaborn 

RUN npm install -g \
	json2csv

# Install Jupyter nbextension
RUN jupyter contrib nbextension install --system --symlink

# Install csvfix
RUN set -x \
	&& curl -fSL "https://bitbucket.org/neilb/csvfix/get/version-1.6.zip" -o /tmp/csvfix.zip \
	&& cd /tmp \
	&& echo "507afcd3fd373182e1de6e5385843f60  csvfix.zip" | md5sum -c \
	&& unzip csvfix.zip \
	&& mv neilb* csvfix \
	&& cd csvfix \
	&& make lin \
	&& mv csvfix/bin/csvfix /usr/local/bin \
	&& rm -rf /tmp/csvfix

# Install csvquote
RUN cd /tmp \
	&& git clone https://github.com/dbro/csvquote.git \
	&& cd csvquote \
	&& make \
	&& make BINDIR=/usr/local/bin/ install \
	&& rm -rf /tmp/csvquote

# Key R packages
RUN echo "install.packages(c('tidyverse', 'devtools', 'ggplot2', 'lubridate', 'data.table'), repos='https://cloud.r-project.org')" \
	| R --slave --no-save --no-restore-history

# Tini for signal forwarding
RUN set -x \
	&& curl -fSL "https://github.com/krallin/tini/releases/download/$TINI_VER/tini" -o /usr/local/bin/tini \
	&& curl -fSL "https://github.com/krallin/tini/releases/download/$TINI_VER/tini.asc" -o /usr/local/bin/tini.asc \
	&& export GNUPGHOME="$(mktemp -d)" \
	# Keyservers are flaky (https://github.com/tianon/gosu/issues/17) -- try again if fails
	&& ( gpg --keyserver hkps://hkps.pool.sks-keyservers.net --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 \
		 || gpg --keyserver pgp.mit.edu --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 \
		 || gpg --keyserver keyserver.pgp.com --recv-keys 595E85A6B1B4779EA4DAAEC70B588DFF0527A9B7 ) \
	&& gpg --batch --verify /usr/local/bin/tini.asc /usr/local/bin/tini \
	&& rm -rf "$GNUPGHOME" /usr/local/bin/tini.asc \
	&& chmod +x /usr/local/bin/tini 

# ------------------------------------------------------------------------------
# Set up user ------------------------------------------------------------------
# ------------------------------------------------------------------------------

# Add user, default shell bash
RUN /usr/sbin/useradd --create-home --shell /bin/bash --no-user-group --uid $CUID $CUSER

# Allow passwordless sudo for CUSER
RUN echo $CUSER' ALL=(ALL) NOPASSWD:ALL' >> /etc/sudoers

# Get dotfiles for user 
# First, create script for working with config file repo (see README in $DOTFILE_REPO)
# Using 'alias' doesn't work, since this is not an interactive shell
RUN echo '#!/bin/bash\n/usr/bin/git --git-dir=$HOME/.myconf/ --work-tree=$HOME "$@"' \
	  > /usr/local/bin/config \
	&& chmod +x /usr/local/bin/config

# Clone config file repo
USER $CUSER

ENV HOME=/home/$CUSER

WORKDIR $HOME

# Cache-busting trick. See https://github.com/moby/moby/issues/1996
# Use for rebuilding image after updating repo by including 
# --build-args CACHEBUST=$(date +%s)
ARG CACHEBUST=1
RUN	git clone --bare $DOTFILE_REPO $HOME/.myconf \
	&& config config --local status.showUntrackedFiles no \
	&& config checkout

# Install Vundle (Vim plugin manager)
RUN git clone https://github.com/VundleVim/Vundle.vim.git ~/.vim/bundle/Vundle.vim

# Install Vim plugins
RUN vim -c 'PluginInstall' -c 'qa!' \
	# Install YouCompleteMe (Vim plugin requiring compile)
	&& cd $HOME/.vim/bundle/YouCompleteMe \
	&& ./install.py

# Move any keys in the build directory to the .ssh/ directory in the image
# Since as of 05/2018 Docker has no conditional copy operation, this uses a trick described
# here http://redgreenrepeat.com/2018/04/13/how-to-conditionally-copy-file-in-dockerfile/
RUN mkdir .ssh
SHELL ["/bin/bash", "-c"]
COPY Dockerfile $KEY_FILE* $HOME/.ssh/
RUN rm -f $HOME/.ssh/Dockerfile \
		&& [[ $(ls -A $HOME/.ssh) ]] \
		&& ( sudo chmod 600 $HOME/.ssh/$KEY_FILE* \
			&& sudo chown -R $CUSER:users $HOME/.ssh ) \
		|| echo "No key found"

# If key(s) present, and $DOTFILE_REPO is github, then configure $DOTFILE_REPO 
# to use SSH (git@github) so that you can push
RUN [[ $(ls -A $HOME/.ssh) && $DOTFILE_REPO = *"github.com"* ]] \
	&& REPO_USER=`echo $DOTFILE_REPO | sed -Ene's#https://github.com/([^/]*)/(.*).git#\1#p'` \
	&& REPO_ADDR=`echo $DOTFILE_REPO | sed -Ene's#https://github.com/([^/]*)/(.*).git#\2#p'` \
	&& [[ -n $REPO_USER && -n $REPO_ADDR ]] \
	&& config remote set-url origin "git@github.com:$REPO_USER/$REPO_ADDR.git" \
	|| echo "No key found"

# Set global name and email options in Git
RUN git config --global user.name $GIT_USER_NAME \
	&& git config --global user.email $GIT_USER_EMAIL

# ------------------------------------------------------------------------------
# Expose port to use with Jupyter ----------------------------------------------
# ------------------------------------------------------------------------------

EXPOSE 8042

ENTRYPOINT ["/usr/local/bin/tini", "--"]

CMD ["/bin/bash"]
