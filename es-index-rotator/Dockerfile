FROM alpine:3.6

COPY rotate.sh /bin/rotate.sh

RUN echo "===> Installing essential tools..."   && \
    apk --update add bash curl coreutils        && \
    echo "===> Cleaning up cache..."            && \
    rm -rf /var/cache/apk/*                     && \
    chmod +x /bin/rotate.sh

CMD ["/bin/rotate.sh"]
