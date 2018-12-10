FROM centos:centos7

# Define versions
ENV pythonVersion=${_PYTHON_VERSION:-3.7.1}
ENV clangVersion=${_CLANG_VERSION:-RELEASE_700}
ENV cmakeVersion=${_CMAKE_VERSION:-3.13.1}
ENV zlibVersion=${_ZLIB_VERSION:-v1.2.11}
ENV libffiVersion=${_LIBFFI_VERSION:-v3.2.1}
ENV makeVersion=${_MAKE_VERSION:-4.2}

ENV bootstrapPrefix=/opt/bootstrap
ENV contribPath=/opt/contrib

ADD https://www.python.org/ftp/python/$pythonVersion/Python-$pythonVersion.tgz Python-$pythonVersion.tgz
ADD https://github.com/Kitware/CMake/releases/download/v$cmakeVersion/cmake-$cmakeVersion-Linux-x86_64.sh cmake-$cmakeVersion.sh
ADD http://ftp.gnu.org/gnu/make/make-4.2.tar.gz make-$makeVersion.tar.gz

# We will install everything under $bootstrapPrefix
ENV PATH="$bootstrapPrefix/bin:$PATH"
RUN echo "export PATH=$bootstrapPrefix/bin:$PATH" > /etc/environment
RUN echo "$bootstrapPrefix/lib" > /etc/ld.so.conf.d/bootstrap.conf && ldconfig

# Some dependencies to bootstrap Python, CMake, Ninja and Clang.
# Extending the following "yum install" line without asking will lend you a kick in the balls.
# If you think that you want to extend this line, often you need to https://www.youtube.com/watch?v=BEpr2T_gKYw&feature=youtu.be&t=8
RUN yum upgrade && yum -y install gcc gcc-c++ glibc-devel subversion git autoconf automake libtool make
# gcc/gcc-c++: 								Needed to compile Python and Clang
# make: 											Needed to build Python
# glibc-devel: 								glibc is hard to link statically so need dynamic linking here. CentOS 7 glibc is really old and using this will lend builds working on distributions using newer glibc versions. This excludes CentOS 6 which is EOL 2020.
# subversion: 								Clone LLVM and Clang
# git: 												Used to clone repositories, not a buildtime or runtime dep so we can use any version we want
# automake/autoconf/libtool		Req to bootstrap libffi, which in turn is needed for Python3
# make 												Req to bootstrap make (this is weird)

# Get, compile and install make (make in CentOS can't build libffi w/o documentation)
RUN tar -xf make-$makeVersion.tar.gz && cd make-$makeVersion && sh ./configure --prefix=$bootstrapPrefix && make && make install && cd ..
# Get, compile and install zlib (Python dep)
RUN git clone https://github.com/libffi/libffi.git libffi && cd libffi && git checkout $libffiVersion && ./autogen.sh && ./configure --disable-docs --prefix=$bootstrapPrefix && make && make install && cd .. && ldconfig
# Get, compile and install libffi (Python dep)
RUN git clone https://github.com/madler/zlib.git zlib && cd zlib && git checkout $zlibVersion && ./configure --prefix=$bootstrapPrefix && make && make install && cd .. && ldconfig
# Compile and install Python (which is also a dep for Ninja and Clang)
RUN tar -xf Python-$pythonVersion.tgz && cd Python-$pythonVersion && ./configure --enable-optimizations --prefix=$bootstrapPrefix && make install && cd ..
# Install CMake (also Clang dep)
RUN chmod +x cmake-$cmakeVersion.sh && ./cmake-$cmakeVersion.sh --prefix=$bootstrapPrefix --skip-license
# Build ninja (Speedup Clang build a bit, as well as future builds)
RUN git clone git://github.com/ninja-build/ninja.git && cd ninja && python3 configure.py --bootstrap && cp ninja $bootstrapPrefix/bin/ && cd ..
# Get Clang
RUN svn co --quiet http://llvm.org/svn/llvm-project/llvm/tags/$clangVersion/final llvm_$clangVersion && cd llvm_$clangVersion/tools && svn co --quiet http://llvm.org/svn/llvm-project/cfe/tags/$clangVersion/final clang && cd ../projects && svn co --quiet http://llvm.org/svn/llvm-project/compiler-rt/tags/$clangVersion/final compiler-rt && svn co --quiet http://llvm.org/svn/llvm-project/libcxx/tags/$clangVersion/final libcxx && svn co --quiet http://llvm.org/svn/llvm-project/libcxxabi/tags/$clangVersion/final libcxxabi && cd ../..
# Build Clang
RUN mkdir llvm_$clangVersion_build && cd llvm_$clangVersion_build && cmake -G Ninja -DCMAKE_BUILD_TYPE=Release -DCMAKE_INSTALL_PREFIX=$bootstrapPrefix ../llvm_$clangVersion && ninja && ninja install && echo "Built Clang successfully!"

# Update ldconfig
RUN ldconfig
