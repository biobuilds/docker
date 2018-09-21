# Repo digest for the 2018-08 "centos:7" image
FROM centos@sha256:6f6d986d425aeabdc3a02cb61c02abb2e78e57357e92417d6d58332856024faf
MAINTAINER BioBuilds Support <support@biobuilds.org>

# Install "lsb_release" script. Also available through the "redhat-lsb-core"
# package, but that would force us to install other things we don't need.
COPY lsb_release /usr/bin/lsb_release

# Make sure a basic set of command-line tools are available
RUN yum install -y \
        bzip2 \
        curl \
        deltarpm \
        file \
        findutils \
        gawk \
        grep \
        gzip \
        hostname \
        openssl \
        sed \
        tar \
        unzip \
        wget \
        which \
        xz \
    && yum clean all

# Install the latest version of Miniconda
ARG MINICONDA_RELEASE=4.5.4
ARG MINICONDA_ROOT=/opt/miniconda
RUN curl -o /tmp/miniconda.sh \
        "https://repo.continuum.io/miniconda/Miniconda3-${MINICONDA_RELEASE}-Linux-`arch`.sh" \
    && /bin/bash /tmp/miniconda.sh -f -b -p "${MINICONDA_ROOT}" \
    && rm -f /tmp/miniconda.sh
ENV PATH ${MINICONDA_ROOT}/bin:$PATH

# Install a basic set of software development tools from the CentOS repository.
#
# NOTE: Starting with the 2018 release(s), BioBuilds recipes should be using
# the Anaconda compilers for all platforms, so we'll skip installing the system
# (i.e., OS vendor) compilers/toolchains into newer build images. This breaks
# backwards compatibility with existing recipes, but the need to rebuild an
# older package is a good sign that we should go ahead and update its recipe.
#
# NOTE: The "man" package provides the `manpath` executable needed by Intel
# Parallel Studio's environment configuration scripts. PSXE is used to build
# "opt"-featured (i.e., "vendor-optimized") linux-64 BioBuilds packages.
RUN yum install -y \
        #binutils \
        #bison \
        #chrpath \
        cvs \
        #flex \
        #gcc \
        #gcc-c++ \
        #gcc-gfortran \
        git \
        #glibc-static \
        #libffi-devel \
        #libstdc++-static \
        make \
        man \
        mercurial \
        #nasm \
        patch \
        perl \
        #pkgconfig \
        svn \
        #vim-common \
    && yum clean all \
    && rm -rf /var/cache/yum

# Previous build images included a minimal set of X11 development packages from
# the CentOS repository, which were needed to build BioBuilds packages that
# depend on things like the cairo vector graphics library. Starting with the
# 2018 releases, we dropped our reliance on these "system" packages in favor of
# their Anaconda-supplied counterparts.
#RUN yum install -y \
#        libX11-devel \
#        libXext-devel \
#        libXrender-devel \
#        libXt-devel \
#    && yum clean all

# Configuration so non-root users can run `conda` in this container
ENV HOME /tmp
COPY condarc /tmp/.condarc
RUN chmod 644 /tmp/.condarc

# Source (i.e., recipes) volume and entrypoint script so BioBuilds packages can
# be built using the recipe repository's `docker-build` script.
VOLUME /src
COPY conda-build.sh /
ENTRYPOINT ["/conda-build.sh"]

# Install updated versions of "conda", "conda-build", and "conda-verify".
#
# Done in two phases (i.e., update "conda" first, then install "conda-build"
# and "conda-verify") to avoid potential problems caused by having too big a
# step from the existing to the desired versions of "conda", such as being
# unable to find the desired "conda-build" and/or "conda-verify" packages.
ARG CONDA_VERSION=4.5.11
ARG CONDA_BUILD_VERSION=3.15.1
ARG CONDA_VERIFY_VERSION=3.1.0
RUN ${MINICONDA_ROOT}/bin/conda install --yes --quiet \
        --override-channels -c defaults \
        conda==${CONDA_VERSION} \
    && echo "conda ==${CONDA_VERSION}" > ${MINICONDA_ROOT}/conda-meta/pinned \
    && ${MINICONDA_ROOT}/bin/conda install --yes --quiet \
        --override-channels -c defaults \
        conda-build==${CONDA_BUILD_VERSION} \
        conda-verify==${CONDA_VERIFY_VERSION} \
    && echo "conda-build ==${CONDA_BUILD_VERSION}" >> ${MINICONDA_ROOT}/conda-meta/pinned \
    && echo "conda-verify ==${CONDA_VERIFY_VERSION}" >> ${MINICONDA_ROOT}/conda-meta/pinned \
    # GNU `tar` ignores root's umask by default, so we need to forcibly remove
    # non-root write permissions from the extracted files. Not doing this can
    # cause `conda` to fail in strange ways when running this image with
    # `docker run -u $UID`. In such cases, the user's primary GID will be 0
    # (root), and `conda` assumes that hard links will work if a package's
    # "index.json" has the group write permission set; however, if some file
    # within a package does *NOT* have group write permissions, hard linking
    # will silently fail, leading to an incomplete package installation within
    # the target conda environment.
    && chmod -R go-w ${MINICONDA_ROOT} \
    # Clean up after ourselves
    && ${MINICONDA_ROOT}/bin/conda clean --packages --tarballs --source-cache --yes
