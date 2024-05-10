FROM --platform=linux/amd64 brineylab/kubeflow-deeplearning

# Labels and maintainer info
LABEL maintainer='Briney Lab @ Scripps Research <briney@scripps.edu>'

# Set user to root for installations
USER root

# Install cmake
RUN apt update \ 
  && apt install -y cmake

# Compile HHsuite from source
RUN git clone --branch v3.3.0 https://github.com/soedinglab/hh-suite.git /tmp/hh-suite \
    && mkdir /tmp/hh-suite/build \
    && pushd /tmp/hh-suite/build \
    && cmake -DCMAKE_INSTALL_PREFIX=/opt/hhsuite .. \
    && make -j 4 && make install \
    && ln -s /opt/hhsuite/bin/* /usr/bin \
    && popd \
    && rm -rf /tmp/hh-suite

# Conda installations 
RUN conda install -y conda==24.1.2 pip python=3.11 \
    && conda install -y -c conda-forge openmm=8.0.0 pdbfixer \
    && conda clean --all --force-pkgs-dirs --yes \
    && fix-permissions "${CONDA_DIR}" \
    && fix-permissions "/home/${NB_USER}"

# Copy AlphaFold files
# FORK alphafold it so we can control the version being installed in the image.
WORKDIR /app/
RUN git clone https://github.com/google-deepmind/alphafold.git
RUN wget -q -P /app/alphafold/alphafold/common/ \
  https://git.scicore.unibas.ch/schwede/openstructure/-/raw/7102c63615b64735c4941278d92b554ec94415f8/modules/mol/alg/src/stereo_chemical_props.txt

# Install pip packages required by AlphaFold and Briney Lab
RUN pip3 install --upgrade pip --no-cache-dir \
    && pip3 install -r /app/alphafold/requirements.txt --no-cache-dir \
    && pip3 install --find-links https://storage.googleapis.com/jax-releases/jax_cuda_releases.html \
      jax==0.4.26 \
      jaxlib==0.4.26+cuda12.cudnn89

# Add SETUID bit to the ldconfig binary so that non-root users can run it.
RUN chmod u+s /sbin/ldconfig.real

# Create an entrypoint script to handle GPU visibility
# WORKDIR /app/alphafold
RUN echo $'#!/bin/bash\nldconfig\npython /app/alphafold/run_alphafold.py "$@"' > /app/run_alphafold.sh \
  && chmod +x /app/run_alphafold.sh
#ENTRYPOINT ["/app/run_alphafold.sh"]

# Setup work directory for backward-compatibility
RUN fix-permissions "/home/${NB_USER}"

# Configure container startup
# ENTRYPOINT ["tini", "-g", "--"]
# CMD ["start.sh"]

# Copy local files as late as possible to avoid cache busting
#COPY start.sh /usr/local/bin/
#RUN chmod +x /usr/local/bin/start.sh
# Switch back to jovyan to avoid accidental container runs as root
USER ${NB_UID}
WORKDIR "${HOME}"
