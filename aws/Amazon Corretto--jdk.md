Create an Image
You can create a new Docker image using Corretto's official Docker Hub image.

Create a Dockerfile with the following content.


FROM amazoncorretto:11
RUN echo $' \
public class Hello { \
public static void main(String[] args) { \
System.out.println("Welcome to Amazon Corretto!"); \
} \
}' > Hello.java
RUN javac Hello.java
CMD ["java", "Hello"] 

Build the new image.


docker build -t hello-app .
Run the new image.
docker run hello-app
You get the following output.

Welcome to Amazon Corretto!