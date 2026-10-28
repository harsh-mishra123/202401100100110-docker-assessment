FROM python:3.13-alpine3.24

WORKDIR /app

# Install dependencies first so this layer is cached between builds.
# pip and setuptools are not needed at runtime and bundle vulnerable libraries.
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt \
 && pip uninstall -y pip setuptools

COPY app.py .

# Run as an unprivileged user instead of root
RUN adduser -S -D -H -u 10001 appuser
USER appuser

ARG APP_VERSION=dev
ENV APP_VERSION=${APP_VERSION}

EXPOSE 5000
CMD ["python", "app.py"]
