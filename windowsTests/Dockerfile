FROM mcr.microsoft.com/windows/servercore:ltsc2019 as prep

ENV PS_VERSION=6.2.7
ADD https://github.com/PowerShell/PowerShell/releases/download/v$PS_VERSION/PowerShell-$PS_VERSION-win-x64.zip /PowerShell/powershell.zip

RUN cd C:\PowerShell &\
    tar.exe -xf powershell.zip &\
    del powershell.zip &\
    mklink powershell.exe pwsh.exe

ADD https://download.sysinternals.com/files/Testlimit.zip /TestLimit/testlimit.zip

RUN cd C:\TestLimit &\
    tar.exe -xf testlimit.zip &\
    del TestLimit.zip


FROM mcr.microsoft.com/windows/nanoserver:1809

COPY --from=prep /PowerShell /PowerShell
COPY --from=prep /TestLimit /TestLimit
COPY ./stress.exe /TestLimit

# set a fixed location for the Module analysis cache
ENV LOCALAPPDATA="C:\Users\ContainerAdministrator\AppData\Local" \
    PSModuleAnalysisCachePath="C:\Users\ContainerAdministrator\AppData\Local\Microsoft\Windows\PowerShell\docker\ModuleAnalysisCache" \
    # Persist %PSCORE% ENV variable for user convenience
    PSCORE="C:\PowerShell\pwsh.exe"

# use downloaded powershell
USER ContainerAdministrator
RUN setx /M PATH "C:\Powershell\;%PATH%"
RUN setx /M PATH "C:\TestLimit\;%PATH%"

# initialize powershell module cache
RUN powershell \
        -NoLogo \
        -NoProfile \
        -Command " \
          $stopTime = (get-date).AddMinutes(15); \
          $ErrorActionPreference = 'Stop' ; \
          $ProgressPreference = 'SilentlyContinue' ; \
          while(!(Test-Path -Path $env:PSModuleAnalysisCachePath)) {  \
            Write-Host "'Waiting for $env:PSModuleAnalysisCachePath'" ; \
            if((get-date) -gt $stopTime) { throw 'timout expired'} \
            Start-Sleep -Seconds 6 ; \
          }"
