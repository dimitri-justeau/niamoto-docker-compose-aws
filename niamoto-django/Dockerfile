FROM python:3.5

MAINTAINER Dimitri Justeau <dimitri.justeau@gmail.com>

# Update sources.list with sid (required for mod_spatialite)
RUN echo 'deb http://httpredir.debian.org/debian sid main' >> /etc/apt/sources.list

# Update and install git
RUN apt-get update && apt-get install -y git && apt-get install sudo && apt-get install daemon

RUN git config --global user.email "niamoto.nc@gmail.com" && \
    git config --global user.name "Niamoto server"

# Add mod_spatialite shared library
RUN apt-get install -y libsqlite3-mod-spatialite libproj-dev gdal-bin libgdal-dev --fix-missing

# Install postgresql-client for dbbackups
RUN sed -e '/deb http\:\/\/httpredir.debian.org\/debian sid main/ s/^#*/#/' -i /etc/apt/sources.list \
    && apt-get update \
    && apt-get install -y postgresql-client --fix-missing \
    && apt-get install -y binutils build-essential

# Install node and browserify
RUN curl -s https://deb.nodesource.com/setup_8.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g browserify \
    && npm install -g yarn

# Install R
RUN sh -c 'echo "deb http://cran.rstudio.com/bin/linux/debian jessie-cran34/" >> /etc/apt/sources.list' \
    && apt-key adv --keyserver keys.gnupg.net --recv-key 'E19F5F87128899B192B1A2C2AD5F960A256A04AF' \
    && apt-get update \
    && apt-get install -y r-base

# Upgrade pip
RUN pip install --upgrade pip

# Create niamoto user, set as current user and set working dir
RUN useradd -m niamoto
ADD sudoers /etc/sudoers
USER niamoto
RUN mkdir -p /home/niamoto
WORKDIR /home/niamoto

# Install Gunicorn
RUN sudo pip install gunicorn
ADD gunicorn_start.sh ./
RUN sudo chmod +x ./gunicorn_start.sh

# Download django-rest-framework
RUN git clone https://github.com/dimitri-justeau/django-rest-framework.git

# Download django-rest-framework-docs
RUN git clone https://github.com/dimitri-justeau/django-rest-framework-docs.git

# Download django-qgis-plugin-repository
RUN git clone https://github.com/dimitri-justeau/django-qgis-plugin-repository.git

# Download niamoto-core
RUN git clone https://github.com/dimitri-justeau/niamoto-core.git

# Download niamoto-portal
RUN git clone https://github.com/niamoto/niamoto-portal.git

# Download niamoto-portal-etl
RUN git clone https://github.com/niamoto/niamoto-portal-etl.git

# Add the niamoto django template setting file
ADD settings.py niamoto-portal/niamoto_portal/

# Add init_superuser script
ADD init_superuser.py /home/niamoto

# Add wait_for_postgres script
ADD wait_for_postgres.py /home/niamoto

# Prepare celery daemon
RUN sudo wget -P /etc/init.d https://raw.githubusercontent.com/celery/celery/3.1/extra/generic-init.d/celeryd
ADD celeryd /etc/default/celeryd

# Prepare flower monitor
ADD flower_startup.sh /home/niamoto/flower_startup.sh
ADD flower /etc/init.d/flower
RUN sudo chmod 0755 /home/niamoto/flower_startup.sh
RUN sudo chmod 0755 /etc/init.d/flower

ADD docker-entrypoint.sh /
RUN sudo chmod +x /docker-entrypoint.sh

EXPOSE 8000
EXPOSE 5555

ENTRYPOINT ["/docker-entrypoint.sh"]
