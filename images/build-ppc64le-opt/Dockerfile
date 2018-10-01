#------------------------------------------------------------------------------
# Dockerfile for BioBuilds build images with the IBM Advance Toolchain (AT)
# installed. Used for building POWER binaries with additional optimizations.
#------------------------------------------------------------------------------
FROM biobuilds/build-ppc64le

# AT 10.0 was the latest release when this file was last updated, but images
# with different releases can be created by passing "--build-args AT_REL=x.y"
# as part of the "docker build" command.
ARG AT_REL=10.0

# AT installation process
COPY ibm-advance-toolchain.repo /etc/yum.repos.d/ibm-advance-toolchain.repo
RUN wget -P /tmp -q "ftp://ftp.unicamp.br/pub/linuxpatch/toolchain/at/redhat/RHEL7/gpg-pubkey-6976a827-5164221b" \
    && rpm --import /tmp/gpg-pubkey-6976a827-5164221b \
    && rm -f /tmp/gpg-pubkey-* \
    && sed -i "s/__AT_REL__/${AT_REL}/g" /etc/yum.repos.d/ibm-advance-toolchain.repo \
    && yum install -y \
        advance-toolchain-at${AT_REL}-devel \
        advance-toolchain-at${AT_REL}-mcore-libs \
        advance-toolchain-at${AT_REL}-runtime \
    && yum clean all \
    && rm -rf /var/cache/yum

# Patch `conda build` to handle AT's special RPATH entries.
COPY conda-build-ibm_at-rpath.patch /tmp/conda-build-ibm_at-rpath.patch
RUN patch -d /opt/miniconda -p0 < /tmp/conda-build-ibm_at-rpath.patch \
    && find /opt/miniconda/lib/python3.6/site-packages -name '*.orig' -print -delete \
    && rm -fv /tmp/*.patch \
    && echo "*** INFO: AT ${AT_REL} install complete!"

# Not setting $PATH as AT is a modified gcc suite and therefore shares the same
# names as the "system" gcc binaries. It's easier to add directories to $PATH
# than to remove them, so we can write simpler conda recipes by detecting
# builds with the "opt" feature enabled and adding AT to $PATH as needed.
#ENV PATH=/opt/at${AT_REL}/bin:${PATH}
