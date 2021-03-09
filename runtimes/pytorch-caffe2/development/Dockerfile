FROM ubuntu:20.04

# Disable interactive installation mode
ENV DEBIAN_FRONTEND=noninteractive

# Install Python dependencies
RUN apt-get update && apt-get install -y \
    build-essential \
    cmake \
    git \
    libgoogle-glog-dev \
    libgtest-dev \
    libgnutls28-dev \
    libleveldb-dev \
    liblmdb-dev \
    libopencv-dev \
    libopenmpi-dev \
    libsnappy-dev \
    libprotobuf-dev \
    openmpi-bin \
    openmpi-doc \
    protobuf-compiler \
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
ENV ONNX_BACKEND="caffe2.python.onnx.backend"

RUN pip3 install onnx

# Temporarily build ONNX with patch due to bug fixed in PR #3256 in onnx/onnx
# After ONNX>1.8.1 is released, remove the following `sed` command:
RUN sed -i 's/defs.get_schema(self.op_type)/defs.get_schema(self.op_type, node.domain)/' \
    /usr/local/lib/python3.8/dist-packages/onnx/backend/test/report/coverage.py 

# Install dependencies
RUN pip3 install --user \
      cmake \
      dataclasses \
      future \
      numpy \
      protobuf \
      typing \
      hypothesis \
      pyyaml
RUN git clone https://github.com/pytorch/pytorch.git && cd pytorch && \
    git submodule update --init --recursive && \
    python3 setup.py install
####################################################

CMD . /root/setup/docker-setup.sh && \
    pytest /root/test/test_backend.py --onnx_backend=${ONNX_BACKEND} -k 'not (_cuda or test_loop11_cpu)' -v
