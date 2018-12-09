FROM centos:centos7

# Define versions
ENV pythonVersion=${VRICON_PYTHON_VERSION:-3.7.1}
ENV clangVersion=${VRICON_CLANG_VERSION:-RELEASE_700}
ENV cmakeVersion=${VRICON_CMAKE_VERSION:-3.13.1}

ENV vriconBootstrapPrefix=/opt/vriconbootstrap
ENV vriconContribPath=/opt/vriconcontrib

ADD https://www.python.org/ftp/python/$pythonVersion/Python-$pythonVersion.tgz Python-$pythonVersion.tgz
ADD https://github.com/Kitware/CMake/releases/download/v$cmakeVersion/cmake-$cmakeVersion-Linux-x86_64.sh cmake-$cmakeVersion.sh

# We will install everything under $vriconBootstrapPrefix
ENV PATH="/opt/vriconbootstrap/bin:$PATH"
RUN echo "export PATH=$vriconBootstrapPrefix/bin:$PATH" > /etc/environment
RUN echo "$vriconBootstrapPrefix/lib" > /etc/ld.so.conf.d/vriconbootstrap.conf && ldconfig

# Some dependencies to bootstrap Python, CMake, Ninja and Clang.
# Extending the following "yum install" line without asking will lend you a kick in the balls.
# If you think that you want to extend this line, often you need to https://www.youtube.com/watch?v=BEpr2T_gKYw&feature=youtu.be&t=8
RUN yum upgrade && yum -y install gcc gcc-c++ make glibc-devel curl xz zlib-devel subversion libffi-devel git
# gcc/gcc-c++: 		Needed to compile Python and Clang
# make: 					Needed to build Python
# glibc-devel: 		glibc is hard to link statically so need dynamic linking here. CentOS 7 glibc is really old and using this will lend builds working on distributions using newer glibc versions. This excludes CentOS 6 which is EOL 2020.
# xz/zlib-devel:	Unpacking stuff for and with Python
# subversion: 		Clone LLVM and Clang
# libffi-devel: 	Fourier transform needed to compile Python
# git: 						Used to clone repositories, not a buildtime or runtime dep so we can use any version we want

# Compile and install Python (which is also a dep for Ninja and Clang)
RUN tar -xf Python-$pythonVersion.tgz && cd Python-$pythonVersion && ./configure --enable-optimizations --prefix=$vriconBootstrapPrefix && make install && cd ..

# Install CMake (also Clang dep)
RUN chmod +x cmake-$cmakeVersion.sh && ./cmake-$cmakeVersion.sh --prefix=$vriconBootstrapPrefix --skip-license

# Build ninja (Speedup Clang build a bit, as well as future builds)
RUN git clone git://github.com/ninja-build/ninja.git && cd ninja && python3 configure.py --bootstrap && cp ninja $vriconBootstrapPrefix/bin/ && cd ..

# Get Clang
RUN svn co --quiet http://llvm.org/svn/llvm-project/llvm/tags/$clangVersion/final llvm_$clangVersion && cd llvm_$clangVersion/tools && svn co --quiet http://llvm.org/svn/llvm-project/cfe/tags/$clangVersion/final clang && cd ../projects && svn co --quiet http://llvm.org/svn/llvm-project/compiler-rt/tags/$clangVersion/final compiler-rt && svn co --quiet http://llvm.org/svn/llvm-project/libcxx/tags/$clangVersion/final libcxx && svn co --quiet http://llvm.org/svn/llvm-project/libcxxabi/tags/$clangVersion/final libcxxabi && cd ../..
# Build Clang
RUN mkdir llvm_$clangVersion_build && cd llvm_$clangVersion_build && cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$vriconBootstrapPrefix ../llvm_$clangVersion && ninja && ninja install && echo "Built Clang successfully!"

# Update ldconfig
RUN ldconfig
