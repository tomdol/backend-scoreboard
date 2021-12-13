FROM ubuntu:20.04

# Disable interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Install Python dependencies
RUN apt-get update && apt-get install -y \
    python3 \
    python3-dev \
    python3-pip && \
    apt-get clean autoclean && apt-get autoremove -y

RUN pip3 install --upgrade pip setuptools wheel

# Copy local directories
COPY ./test /root/test
COPY ./setup /root/setup

# Install test report dependencies
RUN pip3 install --no-cache-dir -r /root/setup/requirements_report.txt

############## ONNX Backend dependencies ###########
ENV ONNX_BACKEND="test_onnx.utils.onnx_backend"

RUN apt-get install -y --no-install-recommends \
    autoconf \
    automake \
    libtool \
    build-essential \
    ninja-build \
    cmake \
    curl \
    cython3 \
    git \
    wget && \
    apt-get clean autoclean && apt-get autoremove -y

# Build OpenVINO
RUN git clone --recurse-submodules --depth 1 https://github.com/openvinotoolkit/openvino.git
WORKDIR /openvino/build
RUN cmake .. \
    -G Ninja \
    -DENABLE_CLDNN=OFF \
    -DENABLE_INTEL_GNA=OFF \
    -DENABLE_OPENCV=OFF \
    -DENABLE_VPU=OFF \
    -DNGRAPH_DEBUG_ENABLE=OFF \
    -DENABLE_TESTS=OFF \
    -DENABLE_BEH_TESTS=OFF \
    -DENABLE_FUNCTIONAL_TESTS=OFF \
    -DENABLE_PROFILING_ITT=OFF \
    -DENABLE_SAMPLES=OFF \
    -DENABLE_MKL_DNN=ON \
    -DENABLE_PYTHON=ON \
    -DPYTHON_EXECUTABLE=/usr/bin/python3 \
    -DCMAKE_INSTALL_PREFIX=/openvino/dist
RUN ninja install

# This hack is needed because OpenVINO conftest.py, which sets up backand name variables,
# is not called while running onnx backend tests.
RUN sed -i 's/BACKEND_NAME = None/BACKEND_NAME = "CPU"/' /openvino/src/bindings/python/tests/__init__.py
# End of hack

WORKDIR /openvino/src/bindings/python
RUN pip3 install -r requirements.txt
RUN pip3 install onnx

RUN source /openvino/dist/setupvars.sh
# ####################################################

CMD . /root/setup/docker-setup.sh && \
    pytest /root/test/test_backend.py --onnx_backend=${ONNX_BACKEND} -k 'not _cuda' -v
