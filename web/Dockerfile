FROM flashspys/nginx-static
RUN apk update && apk upgrade
COPY . /static
RUN rm -rf /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d/default.conf