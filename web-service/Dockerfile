FROM python:3.9-slim

# Set working directory
WORKDIR /app

# Copy requirements.txt and install dependencies
COPY requirements.txt .
RUN pip install -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the port that the Flask app runs on
EXPOSE 80

# Run the application
CMD ["python", "app.py"]


