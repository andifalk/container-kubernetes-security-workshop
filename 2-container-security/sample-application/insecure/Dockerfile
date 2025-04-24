FROM bellsoft/liberica-openjre-alpine:21.0.6
ARG TARGETPLATFORM
RUN echo "I'm building for $TARGETPLATFORM"
COPY ../target/sample-application-1.0.0-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT java -jar /app.jar
