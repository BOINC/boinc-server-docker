# the tag this Dockerfile will build, either "-b2d" or ""
ARG TAG

#===============================
FROM debian:stable-slim AS base
#===============================

LABEL maintainer="Marius Millea <mariusmillea@gmail.com>"

# install packages
RUN apt-get update \
    && apt-get install -y --no-install-recommends \
        ca-certificates \
        curl \
        dh-autoreconf \
        g++ \
        git \
        libcurl4-gnutls-dev \
        default-libmysqlclient-dev \
        libssl-dev \
        libzip-dev \
        m4 \
        make \
        default-mysql-client \
        php8.2-cli \
        php8.2-mysql \
        php8.2-xml \
        pkg-config \
        python3 \
        python3-dev \
        python3-pip \
        python-is-python3 \
        nlohmann-json3-dev \
        libfmt-dev \
        dh-autoreconf \
        libmariadb-dev \
    && rm -rf /var/lib/apt/lists

RUN python3 -m pip install mysqlclient --break-system-packages
# get source and compile server
COPY --chown=1000 boinc /usr/local/boinc
RUN cd /usr/local/boinc && ./_autosetup && ./configure --disable-client --disable-manager && make

# project-making scripts
COPY makeproject-step1.sh makeproject-step2.sh /usr/local/bin/

# some other project files (some of which will be put in the correct place with
# ONBUILD instructions later)
COPY --chown=1000 db_dump_spec.xml /.project_root/
COPY --chown=1000 html /.project_root/html/
COPY --chown=1000 secrets.env /run/secrets/

#==============================
FROM debian:bookworm-slim AS b2d
#==============================

#====================
FROM base AS base-b2d
#====================

#===========================
FROM base$TAG AS makeproject
#===========================

# everything which depends on build-args is done as ONBUILD in this stage, so
# the user can customize it

ARG TAG
ENV TAG=$TAG
ONBUILD ARG BOINC_USER
ONBUILD ARG PROJECT_ROOT
ONBUILD ENV BOINC_USER=$BOINC_USER \
            PROJECT_ROOT=$PROJECT_ROOT \
            USER=$BOINC_USER \
            HOME=/home/$BOINC_USER \
            MYSQL_HOST=mysql

# set up the non-root user who compiles the server
ONBUILD RUN adduser $BOINC_USER --disabled-password --gecos ""

# ensure the project/secrets volumes have the right permissions when mounted
ONBUILD RUN mkdir $PROJECT_ROOT.dst && chown $BOINC_USER:$BOINC_USER $PROJECT_ROOT.dst

ONBUILD USER $BOINC_USER


# build server
ARG CACHEBUST=0
ONBUILD RUN makeproject-step1.sh
ONBUILD CMD makeproject-step2.sh
ONBUILD WORKDIR $PROJECT_ROOT

#==========================================
FROM makeproject AS makeproject-defaultargs
#==========================================

# this triggers the ONBUILD directives using the default ARGs so we also get a
# fully built example image
