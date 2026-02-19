# -------- Stage 1: Build --------
FROM maven:3.9.6-eclipse-temurin-11 AS build

WORKDIR /app
COPY pom.xml .
RUN mvn dependency:go-offline
COPY src ./src
RUN mvn clean package -DskipTests

# -------- Stage 2: Run --------
FROM eclipse-temurin:11-jre

WORKDIR /app
COPY --from=build /app/target/*.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","app.jar"]


# Commands for building and running the Docker container:
#
# cd IdeaProjects/Employee-Management/backend
# (Fix the path to your project directory)
#
# docker build --platform linux/amd64 -t hoangsonw/employee-management-app .
# This builds the Docker image ready for Render deployment.
# docker run -d -p 8080:8080 --env-file config.properties employee-backend
# OR
# docker run -p 8080:8080 \
  #  -e MYSQL_HOST=mysql-host \
  #  -e MYSQL_PORT=3306 \
  #  -e MYSQL_DB=your-db \
  #  -e MYSQL_USER=root \
  #  -e MYSQL_PASSWORD=your-password \
  #  hoangsonw/employee-management-app
# This runs the Docker container locally. Replace the environment variables with your own MySQL database details.
#
# docker push hoangsonw/employee-management-app
# This pushes the Docker image to Docker Hub.
