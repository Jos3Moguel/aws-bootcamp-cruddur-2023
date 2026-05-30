FROM python:3.10-slim-buster

# Inside Container
# Make a new folder inside the container
WORKDIR /backend-flask

# Outside Container -> Inside Container
# This contains the libraries we want to install to run the app
COPY requirements.txt requirements.txt

# Inside Container
# Install the python libraries used for the app
RUN pip3 install -r requirements.txt

# Outside Container -> Inside Container
# . everything in the current directory
# first period . /backend-flask (outside container)
# second period . /backend-flask (inside container)
COPY . .

# Set environment variables
# Inside Container and will remain set when the container is running
ENV FLASK_ENV=development

EXPOSE ${PORT}

# CMD (Command)
# python3 -m flask run --host=0.0.0.0 --port=4567
# -m flask: specifies to use the flask module
CMD [ "python3", "-m" , "flask", "run", "--host=0.0.0.0", "--port=4567"]