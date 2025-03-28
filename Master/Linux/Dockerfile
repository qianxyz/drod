FROM ubuntu:22.04 AS build

RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    git \
    autoconf \
    ninja-build \
    pkg-config \
    libsdl2-dev \
    libsdl2-mixer-dev \
    libsdl2-ttf-dev \
    libtheora-dev \
    libvorbis-dev \
    libogg-dev \
    libpng-dev \
    libjpeg-dev \
    libfreetype6-dev \
    libcurl4-openssl-dev \
    zlib1g-dev \
    libjsoncpp-dev \
    libexpat1-dev \
    wget \
    ca-certificates \
    file \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /build

COPY . .

# Build metakit
WORKDIR /build/metakit/builds
RUN sh ../unix/configure --disable-shared && make
ENV mk_include=/build/metakit/include
ENV mk_lib=/build/metakit/builds

# Build drod
WORKDIR /build/Master/Linux
RUN ./ninjamaker -64 -release -no-static
RUN CPATH="$mk_include" LIBRARY_PATH="$mk_lib" ./build

# Change directory for packaging
WORKDIR /package

# Install linuxdeploy
RUN wget https://github.com/linuxdeploy/linuxdeploy/releases/download/continuous/linuxdeploy-x86_64.AppImage \
    && chmod +x linuxdeploy-x86_64.AppImage

# Make AppDir
RUN mkdir -p AppDir/usr/bin \
    && cp /build/Master/Linux/builds/custom.release.x86_64/drod AppDir/usr/bin/ \
    && wget https://placehold.co/256?text=drod -O AppDir/drod.png
COPY <<'EOT' AppDir/AppRun
#!/bin/sh
HERE="$(dirname "$(readlink -f "$0")")"
export LD_LIBRARY_PATH="${HERE}/usr/lib:${LD_LIBRARY_PATH}"
exec "${HERE}/usr/bin/drod" "$@"
EOT
RUN chmod +x AppDir/AppRun
COPY <<'EOT' AppDir/drod.desktop
[Desktop Entry]
Name=drod-tss-5.2
Exec=drod
Type=Application
Categories=Game;
Icon=drod
EOT

# Create AppImage
RUN ./linuxdeploy-x86_64.AppImage --appimage-extract-and-run \
    --appdir=AppDir \
    --output=appimage

# Export binary
FROM scratch
COPY --from=build /package/drod-tss-5.2-x86_64.AppImage /
