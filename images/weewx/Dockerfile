FROM alpine:3.19
MAINTAINER Rich Braun "docker@instantlinux.net"
ARG BUILD_DATE
ARG VCS_REF
LABEL org.label-schema.build-date=$BUILD_DATE \
    org.label-schema.license=GPL-3.0 \
    org.label-schema.name=weewx \
    org.label-schema.vcs-ref=$VCS_REF \
    org.label-schema.vcs-url=https://github.com/instantlinux/docker-tools

ENV AIRLINK_HOST= \
    ALTITUDE="100, foot" \
    LATITUDE=50.00 \
    LONGITUDE=-80.00 \
    COMPUTER_TYPE="unbranded PC" \
    DB_BINDING_SUFFIX=mysql \
    DB_DRIVER=weedb.mysql \
    DB_HOST=db \
    DB_NAME=weewx_a \
    DB_NAME_FORECAST=weewx_f \
    DB_USER=weewx \
    DEBUG=0 \
    DEVICE_PORT=/dev/ttyUSB0 \
    HTML_ROOT=/var/www/weewx \
    LOCATION="Anytown, USA" \
    LOGGING_INTERVAL=300 \
    OPERATOR="Al Roker" \
    OPTIONAL_ACCESSORIES=False \
    RAIN_YEAR_START=7 \
    RAPIDFIRE=True \
    RSYNC_HOST=web01 \
    RSYNC_PORT=22 \
    RSYNC_DEST=/usr/share/nginx/html \
    RSYNC_USER=wx \
    SKIN=Standard \
    STATION_FEATURES="fan-aspirated shield" \
    STATION_ID=unset \
    STATION_MODEL=6152 \
    STATION_TYPE=Vantage \
    STATION_URL= \
    SYSLOG_DEST=/var/log/messages \
    TZ=US/Eastern \
    TZ_CODE=10 \
    WEBCAM_URL=https://www.wunderground.com/wundermap?lat=37.761&lon=-122.435&zoom=5&radar=1&cam=1 \
    WEEK_START=6 \
    WX_USER=weewx \
    XTIDE_LOCATION=unset

ARG WEEWX_VERSION=5.0.2
ARG WEEWX_SHA=3e55fdf359b9aa3388a1826f0bfad6665a1a865ddfa521286449bcad036c4262
ARG WEEGREEN_VERSION=v0.12
ARG WX_GROUP=dialout
ARG WX_UID=2071
ARG XTIDE_SHA=e5c4afbb17269fdde296e853f2cb84845ed1c1bb1932f780047ad71d623bc681

COPY install-input.txt requirements.txt /root/
RUN apk add --no-cache --update \
      curl freetype libjpeg libstdc++ openssh openssl python3 py3-cheetah \
      py3-configobj py3-mysqlclient py3-pillow py3-requests py3-six py3-usb \
      rsync rsyslog tzdata && \
    adduser -u $WX_UID -s /bin/sh -G $WX_GROUP -D $WX_USER && \
    mkdir build && cd build && \
    curl -sLo weewx.tgz \
      http://www.weewx.com/downloads/released_versions/weewx-$WEEWX_VERSION.tgz && \
    echo "$WEEWX_SHA  weewx.tgz" >> /build/checksums && \
    sha256sum -c /build/checksums && \
    apk add --no-cache --virtual .fetch-deps \
      file freetype-dev g++ gawk gcc git jpeg-dev libpng-dev make musl-dev \
      py3-pip py3-wheel python3-dev zlib-dev && \
    #
    # Lots of breaking changes since 4.x, alas: ticket is SYS-623.
    #   See https://github.com/weewx/weewx/wiki/v5-upgrade
    #   The setup.py procedure found in 4.x went missing in 5.0
    #
    # TODO deal with break-system-packages, no idea what broke
    pip install -r /root/requirements.txt --break-system-packages && \
    tar xf weewx.tgz --strip-components=1 && \
    ### TODO 5/20/2024 version 5.0 no longer has setup.py. Build is broken.
    cd /build && \
    ./setup.py build && ./setup.py install < /root/install-input.txt && \
    git clone -b $WEEGREEN_VERSION --depth 1 \
      https://github.com/instantlinux/weewx-WeeGreen.git \
      /home/$WX_USER/skins/WeeGreen && \
    curl -sLo /tmp/weewx-airlink.zip \
      https://github.com/chaunceygardiner/weewx-airlink/archive/master.zip && \
    /home/$WX_USER/bin/wee_extension --install /tmp/weewx-airlink.zip && \
    apk del .fetch-deps && \
    rm -fr /build /home/$WX_USER/weewx.conf.2* /home/$WX_USER/docs \
      /home/$WX_USER/skins/WeeGreen/.git \
      /root/.cache /var/cache/apk/* /var/log/* /tmp/* && \
    find /home/$WX_USER/bin -name '*.pyc' -exec rm '{}' +;

COPY entrypoint.sh /usr/local/bin
ENTRYPOINT ["/usr/local/bin/entrypoint.sh"]
