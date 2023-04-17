FROM alpine
RUN cat /dev/urandom | tr -dc "[:alnum:]" | fold -w 32 | head -1 > /flag.txt
RUN rm /flag.txt
