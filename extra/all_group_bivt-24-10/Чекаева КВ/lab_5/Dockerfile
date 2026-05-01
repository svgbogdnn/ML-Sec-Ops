FROM ket9/otus-devsecops-xss:latest



RUN groupadd -r appuser && useradd -r -g appuser appuser


USER appuser

EXPOSE 8080

CMD ["sh"]