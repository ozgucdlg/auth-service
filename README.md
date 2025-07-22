# Authentication Service

A robust Spring Boot-based authentication service that provides user registration, login, token management, and authentication verification. Built with AWS DynamoDB for user storage and Redis for token management.

## 🚀 Features

- **User Registration & Login**: Secure user authentication with password hashing
- **Token-Based Authentication**: JWT-style tokens with refresh mechanism
- **Token Management**: Redis-based token storage with automatic expiration
- **Token Verification**: Endpoint to validate authentication tokens
- **AWS Integration**: DynamoDB for user data persistence
- **Scalable Architecture**: Microservice-ready design

## 🛠️ Technology Stack

- **Framework**: Spring Boot 3.5.3
- **Language**: Java 17
- **Database**: AWS DynamoDB
- **Cache**: Redis/ElastiCache
- **Build Tool**: Maven
- **Security**: SHA-256 password hashing

## 📋 Prerequisites

Before running this application, ensure you have:

- **Java 17** or higher
- **Maven** 3.6+
- **Redis** server (local or AWS ElastiCache)
- **AWS Account** with DynamoDB access
- **AWS Credentials** configured

## 🏗️ Project Structure

```
auth-service/
├── auth/
│   ├── src/main/java/com/customer/auth/
│   │   ├── AuthApplication.java          # Main application class
│   │   ├── controller/
│   │   │   └── AuthenticationController.java  # REST endpoints
│   │   ├── service/
│   │   │   ├── AuthenticationService.java     # Business logic
│   │   │   └── TokenService.java              # Token management
│   │   ├── repository/
│   │   │   └── UserRepository.java            # Data access layer
│   │   ├── model/
│   │   │   ├── User.java                      # User entity
│   │   │   └── Token.java                     # Token model
│   │   └── config/
│   │       ├── DynamoDBConfig.java            # DynamoDB configuration
│   │       └── RedisConfig.java               # Redis configuration
│   └── src/main/resources/
│       └── application.properties             # Application configuration
```

## ⚙️ Configuration

### 1. Application Properties

Update `auth/src/main/resources/application.properties`:

```properties
spring.application.name=auth

# Redis Configuration
spring.redis.host=your-redis-host
spring.redis.port=6379

# Server Configuration
server.port=8080
```

### 2. AWS Configuration

Ensure your AWS credentials are configured via:
- AWS CLI: `aws configure`
- Environment variables: `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`
- IAM roles (if running on EC2)

### 3. DynamoDB Setup

Create the required DynamoDB table:

```bash
aws dynamodb create-table \
    --table-name user \
    --attribute-definitions AttributeName=username,AttributeType=S \
    --key-schema AttributeName=username,KeyType=HASH \
    --billing-mode PAY_PER_REQUEST \
    --region eu-central-1
```

## 🚀 Running the Application

### Option 1: Using Maven Wrapper

```bash
cd auth
./mvnw spring-boot:run
```

### Option 2: Using Maven

```bash
cd auth
mvn spring-boot:run
```

### Option 3: Building and Running JAR

```bash
cd auth
mvn clean package
java -jar target/auth-0.0.1-SNAPSHOT.jar
```

The application will start on `http://localhost:8080`

## 📚 API Documentation

### Base URL
```
http://localhost:8080
```

### Endpoints

#### 1. Register User
```http
POST /register
Content-Type: application/json

{
    "username": "string",
    "password": "string"
}
```

**Response (200 OK):**
```json
{
    "token": "access-token-uuid",
    "refToken": "refresh-token-uuid",
    "username": "string"
}
```

#### 2. Login User
```http
POST /login
Content-Type: application/json

{
    "username": "string",
    "password": "string"
}
```

**Response (200 OK):**
```json
{
    "token": "access-token-uuid",
    "refToken": "refresh-token-uuid",
    "username": "string"
}
```

#### 3. Refresh Token
```http
POST /refresh
Content-Type: application/json

{
    "token": "current-access-token",
    "refToken": "current-refresh-token",
    "username": "string"
}
```

**Response (200 OK):**
```json
{
    "token": "new-access-token-uuid",
    "refToken": "new-refresh-token-uuid",
    "username": "string"
}
```

#### 4. Verify Token
```http
GET /verify?token=access-token-uuid
```

**Response (200 OK):**
```
Token is valid
```

## 🔧 Testing

### Using curl

#### Register a new user:
```bash
curl -X POST http://localhost:8080/register \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123"}'
```

#### Login:
```bash
curl -X POST http://localhost:8080/login \
  -H "Content-Type: application/json" \
  -d '{"username":"testuser","password":"password123"}'
```

#### Verify token:
```bash
curl -X GET "http://localhost:8080/verify?token=your-token-here"
```

### Using Postman

Import the following collection:

```json
{
  "info": {
    "name": "Auth Service API",
    "schema": "https://schema.getpostman.com/json/collection/v2.1.0/collection.json"
  },
  "item": [
    {
      "name": "Register User",
      "request": {
        "method": "POST",
        "header": [{"key": "Content-Type", "value": "application/json"}],
        "url": "http://localhost:8080/register",
        "body": {
          "mode": "raw",
          "raw": "{\"username\":\"testuser\",\"password\":\"password123\"}"
        }
      }
    },
    {
      "name": "Login User",
      "request": {
        "method": "POST",
        "header": [{"key": "Content-Type", "value": "application/json"}],
        "url": "http://localhost:8080/login",
        "body": {
          "mode": "raw",
          "raw": "{\"username\":\"testuser\",\"password\":\"password123\"}"
        }
      }
    },
    {
      "name": "Verify Token",
      "request": {
        "method": "GET",
        "url": "http://localhost:8080/verify?token={{token}}"
      }
    }
  ]
}
```

## 🔍 Troubleshooting

### Common Issues

#### 1. Redis Connection Error
```
Network is unreachable: no further information
```
**Solution**: 
- Ensure Redis server is running
- Check Redis host/port configuration
- Verify network connectivity

#### 2. DynamoDB Connection Error
```
Unable to load AWS credentials
```
**Solution**:
- Configure AWS credentials
- Verify DynamoDB table exists
- Check AWS region configuration

#### 3. 500 Internal Server Error
**Common causes**:
- Missing Redis connection
- DynamoDB table not found
- AWS credentials not configured

### Debug Mode

Enable debug logging by adding to `application.properties`:
```properties
logging.level.com.customer.auth=DEBUG
logging.level.org.springframework.data.redis=DEBUG
logging.level.com.amazonaws=DEBUG
```

## 🔒 Security Considerations

- **Password Hashing**: Passwords are hashed using SHA-256
- **Token Expiration**: Access tokens expire after 30 minutes
- **Refresh Tokens**: Valid for 8 hours
- **Token Invalidation**: Old tokens are invalidated on refresh

## 🚀 Deployment

### AWS Deployment

1. **Build the application**:
   ```bash
   mvn clean package
   ```

2. **Deploy to AWS**:
   - Use AWS Elastic Beanstalk
   - Or deploy to EC2 with Docker
   - Configure environment variables for Redis and DynamoDB

### Docker Deployment

Create a `Dockerfile`:
```dockerfile
FROM openjdk:17-jdk-slim
COPY target/auth-0.0.1-SNAPSHOT.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java","-jar","/app.jar"]
```

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Make your changes
4. Add tests
5. Submit a pull request

## 📄 License

This project is licensed under the MIT License.

## 📞 Support

For issues and questions:
- Create an issue in the repository
- Contact the development team

---

**Note**: This is a development version. For production use, consider implementing additional security measures such as JWT tokens, rate limiting, and comprehensive input validation.