FROM openjdk:11.0.12

ENV SCALA_VERSION 2.12.14
ENV SBT_VERSION 1.5.5
ENV USER_ID 1001
ENV GROUP_ID 1001

# Install curl
RUN \
  apt-get update && \
  apt-get -y --no-install-recommends install curl

# Install sbt
RUN \
  curl -fL "https://github.com/sbt/sbt/releases/download/v$SBT_VERSION/sbt-$SBT_VERSION.tgz" | tar xfz - -C /usr/share && \
  chown -R root:root /usr/share/sbt && \
  chmod -R 755 /usr/share/sbt && \
  ln -s /usr/share/sbt/bin/sbt /usr/local/bin/sbt

# Install Scala
## Piping curl directly in tar
RUN \
  case $SCALA_VERSION in \
    "3"*) URL=https://github.com/lampepfl/dotty/releases/download/$SCALA_VERSION/scala3-$SCALA_VERSION.tar.gz SCALA_DIR=/usr/share/scala3-$SCALA_VERSION ;; \
    *) URL=https://downloads.typesafe.com/scala/$SCALA_VERSION/scala-$SCALA_VERSION.tgz SCALA_DIR=/usr/share/scala-$SCALA_VERSION ;; \
  esac && \
  curl -fL $URL | tar xfz - -C /usr/share && \
  mv $SCALA_DIR /usr/share/scala && \
  chown -R root:root /usr/share/scala && \
  chmod -R 755 /usr/share/scala && \
  ln -s /usr/share/scala/bin/* /usr/local/bin
  # case $SCALA_VERSION in \
    # "3"*) echo "@main def main = println(util.Properties.versionMsg)" > test.scala ;; \
    # *) echo "println(util.Properties.versionMsg)" > test.scala ;; \
  # esac && \
  # scala test.scala && rm test.scala

# Remove curl
RUN \
  apt-get -y purge curl && \
  apt-get clean

 # Add and use user sbtuser
 RUN groupadd --gid $GROUP_ID sbtuser && useradd --gid $GROUP_ID --uid $USER_ID sbtuser --shell /bin/bash
 USER sbtuser
 
 # Switch working directory
 WORKDIR /home/sbtuser  

 # Prepare sbt (warm cache)
 RUN \
   sbt sbtVersion && \
   mkdir -p project && \
   echo "scalaVersion := \"${SCALA_VERSION}\"" > build.sbt && \
   echo "sbt.version=${SBT_VERSION}" > project/build.properties && \
   echo "case object Temp" > Temp.scala && \
   sbt compile && \
   rm -r project && rm build.sbt && rm Temp.scala && rm -r target
 
 # Link everything into root as well
 # This allows users of this container to choose, whether they want to run the container as sbtuser (non-root) or as root
 USER root
 RUN \
   ln -s /home/sbtuser/.cache /root/.cache && \
   ln -s /home/sbtuser/.sbt /root/.sbt && \
   if [ -d "/home/sbtuser/.ivy2" ]; then ln -s /home/sbtuser/.ivy2 /root/.ivy2; fi
 
 # Switch working directory back to root
 ## Users wanting to use this container as non-root should combine the two following arguments
 ## -u sbtuser
 ## -w /home/sbtuser
 WORKDIR /root  
 
 CMD sbt
