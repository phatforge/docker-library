FROM ubuntu:14.10
MAINTAINER Pritesh Mehta <pritesh@phatforge.com>

# Install percona repo key and setup apt source
RUN apt-get install -y software-properties-common
RUN apt-key adv --keyserver keys.gnupg.net --recv-keys 1C4CBDCDCD2EFD2A ; \
       echo "deb http://repo.percona.com/apt utopic main" > \
         /etc/apt/sources.list.d/percona.list ; \
       echo "deb-src http://repo.percona.com/apt utopic main" >> \
         /etc/apt/sources.list.d/percona.list ; \
         add-apt-repository ppa:chris-lea/node.js ;

# Install ruby deps
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get install -y \
       git curl ca-certificates wget vim ssh \
       autoconf bison build-essential \
       libssl-dev libyaml-dev libreadline6-dev zlib1g-dev libncurses5-dev \
       libffi-dev libgdbm3 libgdbm-dev

# Upgrade os packages
RUN apt-get update && DEBIAN_FRONTEND=noninteractive apt-get upgrade -y

ENV RUBY_MAJOR 2.0
ENV RUBY_VERSION 2.0.0-p598

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

# skip installing gem documentation
RUN echo 'gem: --no-document' >> "$HOME/.gemrc"

# install things globally, for great justice
ENV GEM_HOME /usr/local/bundle
ENV PATH $GEM_HOME/bin:$PATH
RUN gem install bundler \
    && bundle config --global path "$GEM_HOME" \
    && bundle config --global bin "$GEM_HOME/bin"

# Install app deps
RUN DEBIAN_FRONTEND=noninteractive apt-get install -y percona-toolkit \
       percona-server-client-5.6

# Cleanup apt
RUN apt-get clean && rm -rf /var/lib/apt/lists/* /tmp/* /var/tmp/*

# throw errors if Gemfile has been modified since Gemfile.lock
RUN bundle config --global frozen 1

# prepare for application source
ONBUILD RUN mkdir -p /srv/app/script/docker
ONBUILD WORKDIR /srv/app

# Run setup script from source code to install package and gem deps
ONBUILD COPY Gemfile /srv/app/
ONBUILD COPY Gemfile.lock /srv/app/
ONBUILD COPY ./script/docker/setup /srv/app/script/docker/
ONBUILD RUN ./script/docker/setup

# Copy in source code
ONBUILD COPY . /srv/app

ENV RAILS_ENV=production
EXPOSE 3000
CMD bundle exec rake assets:precompile && foreman start -f Procfile
