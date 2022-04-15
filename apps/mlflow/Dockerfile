# Set the build-related arguments (with default values) for later use
ARG http_proxy=http://proxy.lbs.alcatel-lucent.com:8000
ARG https_proxy=http://proxy.lbs.alcatel-lucent.com:8000
ARG no_proxy=.bell-labs.com
FROM python:3.8.7-slim

ADD requirements.txt /tmp/requirements.txt
RUN pip install -r /tmp/requirements.txt

ENTRYPOINT ["mlflow", "server"]
