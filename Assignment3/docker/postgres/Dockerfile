FROM   postgres:latest

ENV LC_ALL en_US.UTF-8
ENV LC_CTYPE en_US.UTF-8
RUN localedef -i ja_JP -c -f UTF-8 -A /usr/share/locale/locale.alias ja_JP.UTF-8
ENV LANG ja_JP.UTF-

# Copy the initialization sql file to the specified directory in the container
COPY ./docker/postgres/init/*.sql /docker-entrypoint-initdb.d/