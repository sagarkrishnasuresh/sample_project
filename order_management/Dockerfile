# Use a lightweight Java runtime as the base image
FROM openjdk:17-jdk-slim

# Set the working directory inside the container
WORKDIR /app

# Copy the built JAR file into the container
COPY target/order-service-0.0.1-SNAPSHOT.jar app.jar

# Expose the application port
EXPOSE 8082

# Command to run the application
CMD ["java", "-jar", "app.jar"]

