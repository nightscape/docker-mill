#
# Scala and mill Dockerfile
#
# https://github.com/mill/docker-mill
#

# Pull base image
ARG BASE_IMAGE_TAG
FROM openjdk:${BASE_IMAGE_TAG:-11.0.16-jdk-oraclelinux8}

# Env variables
ARG SCALA_VERSION
ENV SCALA_VERSION ${SCALA_VERSION:-2.13.9}
ARG MILL_VERSION
ENV MILL_VERSION ${MILL_VERSION:-0.10.8}
ARG USER_ID
ENV USER_ID ${USER_ID:-1001}
ARG GROUP_ID
ENV GROUP_ID ${GROUP_ID:-1001}

# Install mill
RUN \
  curl -L -o /usr/local/bin/mill https://github.com/lihaoyi/mill/releases/download/$MILL_VERSION/$MILL_VERSION && \
  chmod ugo+x /usr/local/bin/mill

# Install Scala
## Piping curl directly in tar
RUN \
  case $SCALA_VERSION in \
    "3"*) URL=https://github.com/lampepfl/dotty/releases/download/$SCALA_VERSION/scala3-$SCALA_VERSION.tar.gz SCALA_DIR=/usr/share/scala3-$SCALA_VERSION ;; \
    *) URL=https://downloads.typesafe.com/scala/$SCALA_VERSION/scala-$SCALA_VERSION.tgz SCALA_DIR=/usr/share/scala-$SCALA_VERSION ;; \
  esac && \
  curl -fsL $URL | tar xfz - -C /usr/share && \
  mv $SCALA_DIR /usr/share/scala && \
  chown -R root:root /usr/share/scala && \
  chmod -R 755 /usr/share/scala && \
  ln -s /usr/share/scala/bin/* /usr/local/bin && \
  case $SCALA_VERSION in \
    "3"*) echo '@main def main = println(s"Scala library version ${dotty.tools.dotc.config.Properties.versionNumberString}")' > test.scala ;; \
    *) echo "println(util.Properties.versionMsg)" > test.scala ;; \
  esac && \
  scala -nocompdaemon test.scala && rm test.scala

# Add and use user milluser
RUN groupadd --gid $GROUP_ID milluser && useradd --gid $GROUP_ID --uid $USER_ID milluser --shell /bin/bash
USER milluser

# Switch working directory
WORKDIR /home/milluser

# Prepare mill (warm cache)
RUN \
  touch build.sc && \
  mill -i resolve _ && \
  rm build.sc

# Link everything into root as well
# This allows users of this container to choose, whether they want to run the container as milluser (non-root) or as root
USER root
RUN \
  rm -rf /tmp/..?* /tmp/.[!.]* * && \
  ln -s /home/milluser/.cache /root/.cache && \
  ln -s /home/milluser/.mill /root/.mill && \
  if [ -d "/home/milluser/.ivy2" ]; then ln -s /home/milluser/.ivy2 /root/.ivy2; fi

# Switch working directory back to root
## Users wanting to use this container as non-root should combine the two following arguments
## -u milluser
## -w /home/milluser
WORKDIR /root

CMD mill
