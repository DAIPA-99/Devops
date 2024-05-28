FROM eclipse-temurin:21-jdk-alpine
COPY Main.java /app/Main.java
WORKDIR /app
RUN javac Main.java
 
FROM eclipse-temurin:21-jre-alpine
COPY --from=0 /app/Main.class .
 
CMD ["java", "Main"]