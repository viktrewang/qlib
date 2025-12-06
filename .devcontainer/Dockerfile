# Development container for Qlib
# Build with: docker build -t qlib-dev .
# Run with: docker-compose up -d

FROM continuumio/miniconda3:latest

LABEL maintainer="Qlib Development"
LABEL description="Development environment for Qlib quantitative research platform"

# Set working directory
WORKDIR /workspace

# Update system packages
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    cmake \
    git \
    curl \
    wget \
    vim \
    nano \
    htop \
    pkg-config \
    libffi-dev \
    libssl-dev \
    && rm -rf /var/lib/apt/lists/*

# Create conda environment
ARG PYTHON_VERSION=3.10
RUN conda create -n qlib python=${PYTHON_VERSION} -y && \
    echo "conda activate qlib" >> ~/.bashrc

# Activate environment for subsequent commands
ENV PATH /opt/conda/envs/qlib/bin:$PATH

# Upgrade pip and install build tools
RUN pip install --upgrade pip setuptools wheel && \
    pip install --upgrade cython

# Install core dependencies
RUN pip install \
    numpy \
    pandas \
    scipy \
    scikit-learn \
    pyyaml \
    fire \
    tqdm \
    joblib \
    matplotlib \
    jupyter \
    jupyterlab \
    notebook \
    ipython \
    ipykernel \
    nbconvert

# Install additional dependencies from pyproject.toml
RUN pip install \
    mlflow \
    filelock \
    redis \
    dill \
    ruamel.yaml \
    python-redis-lock \
    pymongo \
    loguru \
    lightgbm \
    gym \
    cvxpy \
    pyarrow \
    pydantic-settings \
    statsmodels

# Optional: Install development tools
RUN pip install \
    pytest \
    pytest-cov \
    black \
    pylint \
    mypy \
    flake8 \
    nbqa \
    ipdb

# Copy project files (including .git for setuptools-scm)
COPY . /workspace/

# Install qlib in editable mode
# Set SETUPTOOLS_SCM_PRETEND_VERSION as fallback if git metadata is missing
ENV SETUPTOOLS_SCM_PRETEND_VERSION=0.9.0.dev0
RUN cd /workspace && pip install -e .

# Configure Jupyter
RUN jupyter notebook --generate-config && \
    echo "c.NotebookApp.ip = '0.0.0.0'" >> ~/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.allow_root = True" >> ~/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.token = ''" >> ~/.jupyter/jupyter_notebook_config.py && \
    echo "c.NotebookApp.password = ''" >> ~/.jupyter/jupyter_notebook_config.py

# Setup git config
RUN git config --global --add safe.directory /workspace

# Create startup script
RUN echo '#!/bin/bash\nset -e\n\necho "Qlib Development Environment Started"\necho "======================================"\necho "Python: $(python --version)"\necho "Qlib: $(python -c \"import qlib; print(qlib.__version__ if hasattr(qlib, \"__version__\") else \"installed\")\" 2>/dev/null || echo \"installation pending\")"\necho ""\necho "Available commands:"\necho "  - python: Run Python scripts"\necho "  - jupyter notebook: Start Jupyter notebook on port 8888"\necho "  - pytest: Run tests"\necho "  - ipython: Start IPython shell"\necho ""\n/bin/bash' > /startup.sh && \
    chmod +x /startup.sh

ENTRYPOINT ["/bin/bash"]
CMD ["-i"]
