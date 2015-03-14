FROM ubuntu:14.10
MAINTAINER Pritesh Mehta <pritesh@phatforge.com>

ENV RUBY_MAJOR 2.0
ENV RUBY_VERSION 2.0.0-p598
ENV APP_ROOT /srv/app

# Install percona repo key and setup apt source
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y software-properties-common
RUN apt-key adv --keyserver keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A ; \
       echo "deb http://repo.percona.com/apt utopic main" > \
         /etc/apt/sources.list.d/percona.list ; \
       echo "deb-src http://repo.percona.com/apt utopic main" >> \
         /etc/apt/sources.list.d/percona.list ; \
         add-apt-repository ppa:chris-lea/node.js ; \
         add-apt-repository ppa:chris-lea/redis-server ;

# Install ruby deps
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get install -y \
       git curl ca-certificates wget vim ssh \
       autoconf bison build-essential \
       libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev \
       libffi-dev libgdbm3 libgdbm-dev

# Upgrade os packages
RUN DEBIAN_FRONTEND=noninteractive apt-get update && apt-get upgrade -y

# Install ruby
RUN mkdir -p /usr/src/ruby \
      && curl -SL "http://cache.ruby-lang.org/pub/ruby/$RUBY_MAJOR/ruby-$RUBY_VERSION.tar.bz2" \
      | tar -xjC /usr/src/ruby --strip-components=1 \
      && cd /usr/src/ruby \
      && autoconf \
      && ./configure --disable-install-doc \
      && make -j"$(nproc)" \
      && apt-get purge -y --auto-remove bison ruby \
      && make install \
      && rm -r /usr/src/ruby

# Remove conflicting rake
RUN DEBIAN_FRONTEND=noninteractive apt-get purge rake libmysqlclient18 \
      libmysqlclient18

# Install rails deps
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y percona-toolkit \
       percona-server-client-5.6 libperconaserverclient18.1-dev

# Cleanup apt
RUN DEBIAN_FRONTEND=noninteractive apt-get -y autoremove && apt-get -y clean \
      && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# skip installing gem documentation
RUN echo 'gem: --no-document' >> "$HOME/.gemrc"

# install things globally, for great justice
RUN gem install bundler

# prepare for application source
RUN mkdir -p /srv/app/script/docker
WORKDIR /srv/app

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

# Run setup script from source code to install package and gem deps
ONBUILD COPY Gemfile ${APP_ROOT}
ONBUILD COPY Gemfile.lock ${APP_ROOT}
ONBUILD COPY ./script/docker/ ${APP_ROOT}/script/docker/
ONBUILD RUN ./script/docker/bootstrap
ONBUILD RUN ./script/docker/bundle_install
ONBUILD RUN ./script/docker/setup

# Copy in source code
ONBUILD COPY . ${APP_ROOT}

ENV RAILS_ENV=production
EXPOSE 5000
CMD bundle exec rake assets:precompile && foreman start -f Procfile
