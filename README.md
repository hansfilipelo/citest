# CI-testing

Simple build system tests with Docker, CMake, MSBuild, Ninja and Python.

# Notes

- VS build tools and docker:

https://docs.microsoft.com/en-us/visualstudio/install/build-tools-container?view=vs-2017

- Building Clang requires more RAM, add to docker build:

```
docker build -m 5GB -t centosbuildtools
```
