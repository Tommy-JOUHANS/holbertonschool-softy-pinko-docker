FROM ubuntu:24.04

RUN apt-get update && apt-get install -y python3 python3-pip
RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true

WORKDIR /app

COPY requirements.txt /app
RUN pip install -r /app/requirements.txt

COPY api.py /app

EXPOSE 5252
CMD ["python3", "api.py"]