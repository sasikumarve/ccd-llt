# Containers and Cloud Devops - LLT
**CCD LLT project**

**Pipeline Design**
Code check-in to "main" branch triggers the workflow.

**Steps involved:**
1. Checkout Code
2. Configure AWS credentials from OIDC
3. Ensure ECR repository exists
4. Login to AWS ECR
5. Build Docker image (Multi-stage Docker build)
6. Tag Docker image
7. Push Docker image to Amazon ECR

**IAM Role Design**

Setup a Role in IAM of AWS with Trust relationships pointing to the following.
1. githubusercontent token
2. github repo path with brancch information
3. AWS Identity provider Audience ID.
   
The IAM role should be able to pull information from github using secure credentials and build an Image in the ECR.

**Challenges faced**

None

**Multi-Stage build: working**

*Stages:*
1. Build the java code using maven "mvn clean package"
2. Run the bundled jar in "eclipse-temurin:17-jdk-alpine" base image.
