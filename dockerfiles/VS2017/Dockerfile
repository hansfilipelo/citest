# escape=`
FROM FROM microsoft/dotnet-framework:4.7.1

# Restore the default Windows shell for correct batch processing below.
SHELL ["cmd", "/S", "/C"]
WORKDIR C:\TEMP

# Download the Build Tools bootstrapper.
ADD https://aka.ms/vs/15/release/vs_buildtools.exe C:\TEMP\vs_buildtools.exe
# Download git, python (required to bootstrap ninja) as well as CMake
ADD https://github.com/git-for-windows/git/releases/download/v2.19.1.windows.1/Git-2.19.1-64-bit.exe C:\TEMP\git-install.exe
ADD https://www.python.org/ftp/python/3.7.1/python-3.7.1.exe C:\TEMP\python-install.exe
ADD https://github.com/Kitware/CMake/releases/download/v3.13.1/cmake-3.13.1-win64-x64.msi C:\TEMP\cmake-install.msi

# Install Build Tools, excluding workloads and components with known issues.
RUN C:\TEMP\vs_buildtools.exe --quiet --wait --norestart --nocache `
    --installPath C:\BuildTools `
    --all `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10240 `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.10586 `
    --remove Microsoft.VisualStudio.Component.Windows10SDK.14393 `
    --remove Microsoft.VisualStudio.Component.Windows81SDK `
 || IF "%ERRORLEVEL%"=="3010" EXIT 0

# Install Git
RUN C:\TEMP\git-install.exe /LOADINF=git.inf /LOG=C:\TEMP\git.log /VERYSILENT && set PATH=%PATH%;"C:\Program Files\Git\bin"

## Install Python and add to path
RUN C:\TEMP\python-install.exe /quiet /log C:\TEMP\python.log InstallAllUsers=1 PrependPath=1 Include_test=0 && set PATH=%PATH%;"C:\Program Files (x86)\Python37-32"

## Install Cmake and add to path
RUN msiexec.exe /i C:\TEMP\cmake-install.msi && set PATH=%PATH%;"C:\Program Files\CMake\bin"

# Build Ninja and add to a place in PATH (copy to git's command folder)
RUN git clone https://github.com/ninja-build/ninja.git
WORKDIR C:\TEMP\ninja
RUN git checkout release
RUN C:\BuildTools\Common7\Tools\VsDevCmd.bat && python configure.py --bootstrap && ninja.bootstrap.exe
RUN copy ninja.exe "C:\Program Files\Git\bin\\"

# Start developer command prompt with any other commands specified.
WORKDIR C:\TEMP
ENTRYPOINT C:\BuildTools\Common7\Tools\VsDevCmd.bat &&

# Default to PowerShell if no other command specified.
CMD ["powershell.exe", "-NoLogo", "-ExecutionPolicy", "Bypass"]
