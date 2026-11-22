# syntax=docker/dockerfile:1

# Contexte de build attendu : le dossier parent contenant
#   ./Radarr/          (ce dépôt)
#   ./docker-radarr/   (https://github.com/linuxserver/docker-radarr, pour root/)
# En local : docker build -f Radarr/.github/docker/Dockerfile .

############################
# Stage 1 : compilation
############################
FROM mcr.microsoft.com/dotnet/sdk:10.0 AS build

RUN apt-get update \
    && apt-get install -y --no-install-recommends curl ca-certificates gnupg \
    && curl -fsSL https://deb.nodesource.com/setup_24.x | bash - \
    && apt-get install -y nodejs \
    && npm install -g yarn \
    && rm -rf /var/lib/apt/lists/*

ENV DOTNET_CLI_TELEMETRY_OPTOUT=1

# Lus par build.sh (UpdateVersionNumber) pour remplir AssemblyVersion / AssemblyConfiguration
ARG RADARRVERSION=""
ARG BUILD_SOURCEBRANCHNAME=""
ENV RADARRVERSION=${RADARRVERSION} \
    BUILD_SOURCEBRANCHNAME=${BUILD_SOURCEBRANCHNAME}

WORKDIR /src
COPY ./Radarr/ .
# global.json épingle une version exacte du SDK, l'image en fournit la dernière 10.0.x
# build.sh fait un find sur _temp/obj et échoue s'il n'existe pas (checkout propre)
RUN rm -f global.json && mkdir -p _temp/obj

RUN --mount=type=cache,target=/root/.nuget/packages \
    --mount=type=cache,target=/usr/local/share/.cache/yarn \
    ./build.sh --backend --frontend -f net10.0 -r linux-musl-x64

############################
# Stage 2 : image finale
############################
FROM ghcr.io/linuxserver/baseimage-alpine:3.24

ARG BUILD_DATE
ARG VERSION
LABEL build_version="Linuxserver.io version:- ${VERSION} Build-date:- ${BUILD_DATE}"
LABEL maintainer="Roxedus,thespad"

ENV XDG_CONFIG_HOME="/config/xdg" \
    RADARR_BRANCH="master" \
    COMPlus_EnableDiagnostics=0 \
    TMPDIR=/run/radarr-temp

RUN apk add --no-cache icu-libs sqlite-libs xmlstarlet

COPY --from=build --chmod=755 /src/_output/net10.0/linux-musl-x64/publish/ /app/radarr/bin/
COPY --from=build /src/_output/UI/ /app/radarr/bin/UI/

RUN printf "UpdateMethod=docker\nBranch=${RADARR_BRANCH}\nPackageVersion=${VERSION:-LocalBuild}\nPackageAuthor=[lasyan3](https://github.com/lasyan3/Radarr)\n" > /app/radarr/package_info \
    && printf "Linuxserver.io version: ${VERSION}\nBuild-date: ${BUILD_DATE}" > /build_version

COPY docker-radarr/root/ /

EXPOSE 7878
VOLUME /config
