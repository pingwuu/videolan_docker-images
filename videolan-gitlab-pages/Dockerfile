FROM debian:buster-20220125-slim

MAINTAINER VideoLAN roots <roots@videolan.org>

ENV IMAGE_DATE=202007231240

# If someone wants to use VideoLAN docker images on a local machine and does
# not want to be disturbed by the videolan user, we should not take an uid/gid
# in the user range of main distributions, which means:
# - Debian based: <1000
# - RPM based: <500 (CentOS, RedHat, etc.)
ARG VIDEOLAN_UID=499

RUN addgroup --quiet --gid ${VIDEOLAN_UID} videolan && \
    adduser --quiet --uid ${VIDEOLAN_UID} --ingroup videolan videolan && \
    usermod --append --groups video videolan && \
    echo "videolan:videolan" | chpasswd && \
    echo "deb http://ftp.fr.debian.org/debian/ buster main" > /etc/apt/sources.list && \
    echo "deb-src http://ftp.fr.debian.org/debian/ buster main" >> /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-suggests --no-install-recommends \
        build-essential zlib1g-dev ruby-full bundler \
        python3-pip python3-setuptools \
	graphviz && \
    pip3 install Sphinx==4.4.0 'sphinx-intl>=2.0.1' sphinx-rtd-theme==1.0.0 sphinx-tabs==3.3.1 && \
    apt-get clean -y && rm -rf /var/lib/apt/lists/*

USER videolan
