FROM postgres:12.1

RUN localedef -i ja_JP -c -f UTF-8 -A /usr/share/locale/locale.alias ja_JP.UTF-8
ENV LANG ja_JP.utf8

COPY docker-entrypoint-initdb.d /docker-entrypoint-initdb.d

CMD ["postgres"]

EXPOSE 5432
