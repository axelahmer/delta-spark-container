# Credit: https://github.com/delta-io/delta-docker/blob/main/Dockerfile_delta_quickstart

ARG BASE_CONTAINER=spark:3.5.1-scala2.12-java17-python3-ubuntu
FROM $BASE_CONTAINER as spark
FROM spark as delta

USER root

# Set environment variables for package versions
ARG DELTA_SPARK_VERSION="3.1.0"
ARG DELTALAKE_VERSION="0.16.4"
ARG JUPYTERLAB_VERSION="4.0.7"
ARG PANDAS_VERSION="2.2.2"

# Install Python packages
RUN pip install --quiet --no-cache-dir \
  delta-spark==${DELTA_SPARK_VERSION} \
  deltalake==${DELTALAKE_VERSION} \
  jupyterlab==${JUPYTERLAB_VERSION} \
  pandas==${PANDAS_VERSION}

# Install git
RUN apt-get update && apt-get install -y git

# Set up work directory
ARG WORKDIR=/opt/spark/work-dir
ENV DELTA_PACKAGE_VERSION=delta-spark_2.12:${DELTA_SPARK_VERSION}

# Create startup script
RUN echo '#!/bin/bash\n\
source "$HOME/.cargo/env"\n\
export PYSPARK_DRIVER_PYTHON=jupyter\n\
export PYSPARK_DRIVER_PYTHON_OPTS="lab --ip=0.0.0.0 --allow-root"\n\
export DELTA_SPARK_VERSION="3.1.0"\n\
export DELTA_PACKAGE_VERSION=delta-spark_2.12:${DELTA_SPARK_VERSION}\n\
exec $SPARK_HOME/bin/pyspark --packages io.delta:${DELTA_PACKAGE_VERSION} \\\n\
--conf "spark.driver.extraJavaOptions=-Divy.cache.dir=/tmp -Divy.home=/tmp -Dio.netty.tryReflectionSetAccessible=true" \\\n\
--conf "spark.sql.extensions=io.delta.sql.DeltaSparkSessionExtension" \\\n\
--conf "spark.sql.catalog.spark_catalog=org.apache.spark.sql.delta.catalog.DeltaCatalog"\n\
' > /startup.sh && chmod +x /startup.sh

RUN curl https://sh.rustup.rs -sSf | sh -s -- -y
# moved the source command into the bash process in the entrypoint startup.sh
#RUN source "$HOME/.cargo/env"

WORKDIR $WORKDIR

# Expose Jupyter port
EXPOSE 8888

ENTRYPOINT ["/startup.sh"]