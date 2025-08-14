# Use the official Python 3.12 "slim" base image
# "slim" variant = smaller image, minimal system packages
FROM python:3.12-slim

# ---------------------------------------------------------
#  ENVIRONMENT VARIABLES CONFIGURATION
# ---------------------------------------------------------

# Set Airflow's home directory inside the container
# Stores DAGs, logs, plugins, and airflow.cfg here
# /opt is a standard Linux path for optional application software
ENV AIRFLOW_HOME=/opt/airflow

# Specify the Apache Airflow version to install
ENV AIRFLOW_VERSION=2.10.3

# Specify the Python version (used in Airflow constraints URL)
ENV PYTHON_VERSION=3.12

# URL for Airflow's official constraints file (specific to version & Python)
# Ensures pip installs dependency versions tested to work with this Airflow release
ENV CONSTRAINT_URL="https://raw.githubusercontent.com/apache/airflow/constraints-${AIRFLOW_VERSION}/constraints-${PYTHON_VERSION}.txt"

# ---------------------------------------------------------
#  INSTALL SYSTEM DEPENDENCIES
# ---------------------------------------------------------
RUN apt-get update && \
    apt-get install -y --no-install-recommends \
        # Tools required for compiling Python packages with native code
        build-essential \
        # Command-line tool to transfer data from URLs
        curl \
        # Version control system for pulling code repositories
        git \
        # PostgreSQL client library headers (needed for psycopg2)
        libpq-dev \
        # Utilities for working with PDFs (pdftotext, pdfimages, etc.)
        poppler-utils \
        # Optical Character Recognition (OCR) tool for extracting text from images
        tesseract-ocr \
        # Remove cached apt lists to reduce image size
        && rm -rf /var/lib/apt/lists/*

# ---------------------------------------------------------
#  CREATE AIRFLOW USER (non-root for security)
# ---------------------------------------------------------
RUN groupadd -r -g 50000 airflow && \
    # Create a system user with:
    #  - UID: 50000 (chosen for cross-platform consistency)
    #  - GID: 50000 (matching the group above)
    #  - Home directory: ${AIRFLOW_HOME}
    #  - Default shell: /bin/bash
    useradd -r -u 50000 -g airflow -d ${AIRFLOW_HOME} -s /bin/bash airflow

# ---------------------------------------------------------
#  CREATE AIRFLOW DIRECTORIES AND SET PERMISSIONS
# ---------------------------------------------------------
RUN mkdir -p ${AIRFLOW_HOME} && \
    # Create DAGs folder (stores workflow definition Python files)
    mkdir -p ${AIRFLOW_HOME}/dags && \
    # Create logs folder (stores task execution logs)
    mkdir -p ${AIRFLOW_HOME}/logs && \
    # Create plugins folder (stores custom Airflow plugins, operators, hooks)
    mkdir -p ${AIRFLOW_HOME}/plugins && \
    # Give ownership of all Airflow files to airflow user/group
    chown -R 50000:50000 ${AIRFLOW_HOME} && \
    # Set directory permissions to allow read/execute for all, write for owner
    chmod -R 755 ${AIRFLOW_HOME}

# ---------------------------------------------------------
#  INSTALL AIRFLOW WITH POSTGRESQL SUPPORT
# ---------------------------------------------------------
RUN pip install --no-cache-dir \
    # Install Apache Airflow with PostgreSQL database support
    "apache-airflow[postgres]==${AIRFLOW_VERSION}" \
    # Apply version constraints to avoid dependency conflicts
    --constraint "${CONSTRAINT_URL}" \
    # Install PostgreSQL Python driver
    psycopg2-binary

# ---------------------------------------------------------
#  INSTALL ADDITIONAL PROJECT DEPENDENCIES
# ---------------------------------------------------------
# Copy custom Airflow-related dependencies file into container
COPY requirements-airflow.txt /tmp/requirements-airflow.txt

# Install dependencies listed in the copied file
RUN pip install --no-cache-dir -r /tmp/requirements-airflow.txt

# ---------------------------------------------------------
#  SETUP ENTRYPOINT SCRIPT
# ---------------------------------------------------------
# Copy entrypoint script from host machine to container root
COPY entrypoint.sh /entrypoint.sh

# Make the entrypoint script executable
RUN chmod +x /entrypoint.sh

# ---------------------------------------------------------
#  SWITCH TO NON-ROOT USER & SET WORKDIR
# ---------------------------------------------------------
# Switch to airflow user for security reasons
USER airflow

# Set working directory to Airflow home
WORKDIR ${AIRFLOW_HOME}

# ---------------------------------------------------------
#  EXPOSE WEB SERVER PORT
# ---------------------------------------------------------
# Airflow webserver listens on port 8080 by default
EXPOSE 8080

# ---------------------------------------------------------
#  DEFAULT CONTAINER COMMAND
# ---------------------------------------------------------
# When container starts, run the entrypoint script
CMD ["/entrypoint.sh"]
