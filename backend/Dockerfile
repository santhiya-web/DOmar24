# Use the official Python image
FROM python:3.11-slim

# Set the working directory
WORKDIR /app

# Copy the requirements file into the container
COPY requirements.txt .

# Install the Python dependencies
RUN pip install --no-cache-dir -r requirements.txt

# Copy the rest of the application code
COPY . .

# Expose the Cloud Run port
EXPOSE 8080

# Command to run the Fastapi application
CMD ["uvicorn", "main:app", "--host", "0.0.0.0", "--port", "8080"]
