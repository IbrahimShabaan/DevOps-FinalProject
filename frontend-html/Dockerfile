FROM python:3.9-slim
RUN apt-get update && apt-get install -y --no-install-recommends \
    build-essential \
    && rm -rf /var/lib/apt/lists/*
RUN mkdir /code
WORKDIR /code
# Copy and install Python dependencies
ADD requirements.txt /code/
RUN pip3 install --no-cache-dir -r requirements.txt
ADD . /code/
EXPOSE 5001
CMD ["python3", "front.py"]
