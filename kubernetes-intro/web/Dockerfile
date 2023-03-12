FROM nginx:latest 
LABEL maintainer="aygul.maltseva@wn.media"

RUN mkdir /app && \
    chown nginx:nginx /app

COPY content/default.conf /etc/nginx/conf.d/
