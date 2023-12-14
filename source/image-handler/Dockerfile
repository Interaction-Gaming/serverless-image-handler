# Define custom function directory
ARG FUNCTION_DIR="/function"

FROM node:16-bookworm

# Include global arg in this stage of the build
ARG FUNCTION_DIR

# Install build dependencies
RUN apt-get update && \
    apt-get install -y \
    g++ \
    make \
    cmake \
    unzip \
    build-essential \
    ninja-build \
    python3-pip \
    meson \
    bc \
    wget \
    libcurl4-openssl-dev \
    libcgif-dev \
    libheif-dev \
    libfftw3-dev \
    libopenexr-dev \
    libgsf-1-dev \
    libglib2.0-dev \
    libglib2.0-0 \
    liborc-dev \
    libopenslide-dev \
    libmatio-dev \
    libwebp-dev \
    libjpeg-dev \
    libexpat1-dev \
    libexif-dev \
    libtiff5-dev \
    libcfitsio-dev \
    libpoppler-glib-dev \
    librsvg2-dev \
    libpango1.0-dev \
    libopenjp2-7-dev \
    libimagequant-dev && \
    apt-get remove libvips42

# Build libvips
RUN wget https://github.com/libvips/libvips/releases/download/v8.15.0/vips-8.15.0.tar.xz && \
    tar xf vips-8.15.0.tar.xz && \
    cd vips-8.15.0 && \
    meson build --libdir=lib --buildtype=release -Dintrospection=disabled && \
    cd build && \
    meson compile && \
    meson test && \
    meson install && \
    echo "/usr/local/lib" > /etc/ld.so.conf.d/usrlocal.conf && \
    ldconfig -v

# Copy function code
RUN mkdir -p ${FUNCTION_DIR}
COPY . ${FUNCTION_DIR}

WORKDIR ${FUNCTION_DIR}

# Install Node.js dependencies
RUN npm ci && rm -rf node_modules/sharp && npm install --arch=x64 --platform=linux sharp

# Install the runtime interface client
RUN npm install aws-lambda-ric

# Required for Node runtimes which use npm@8.6.0+ because
# by default npm writes logs under /home/.npm and Lambda fs is read-only
ENV NPM_CONFIG_CACHE=/tmp/.npm

# Include global arg in this stage of the build
ARG FUNCTION_DIR

# Set working directory to function root directory
WORKDIR ${FUNCTION_DIR}

# Set runtime interface client as default command for the container runtime
ENTRYPOINT ["/usr/local/bin/npx", "aws-lambda-ric"]
# Pass the name of the function handler as an argument to the runtime
CMD ["dist/image-handler/index.handler"]