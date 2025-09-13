# Use latest Ubuntu as base image
FROM ubuntu:latest

# Update and Upgrade
RUN apt-get update && apt-get upgrade -y

# Install Python3 and pip3
RUN apt-get install -y python3 python3-pip

# Fix EXTERNALLY-MANAGED error 
RUN rm /usr/lib/python*/EXTERNALLY-MANAGED || true

# Copy requirements 
COPY requirements.txt . 

# Install dependencies from requirements.txt
RUN pip3 install --no-cache-dir -r requirements.txt

# Set working directory
WORKDIR /app

# Copy app code into container     
COPY ./api.py /app/api.py

# Expose port 5252
EXPOSE 5252

# Run the Flask app
CMD ["python3", "api.py"]