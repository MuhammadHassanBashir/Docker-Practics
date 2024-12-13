## Deploying sample application on instance using docker compose and connecting application to aws RDS

STEPS
-----

## 1. Directory Structure

    project/
    ├── app/
    │   ├── app.py           # Hello World application code
    │   ├── requirements.txt # Python dependencies
    ├── .env                 # Environment variables
    ├── Dockerfile           # Docker build instructions
    └── docker-compose.yml   # Docker Compose configuration

## 2. Code: Application Code (app/app.py)

    from flask import Flask
    import os
    import pymysql
    
    app = Flask(__name__)
    
    # Environment variables
    db_host = os.getenv("DB_HOST")
    db_user = os.getenv("DB_USER")
    db_password = os.getenv("DB_PASSWORD")
    db_name = os.getenv("DB_NAME")
    
    @app.route("/")
    def hello_world():
        try:
            # Connecting to AWS RDS
            connection = pymysql.connect(
                host=db_host,
                user=db_user,
                password=db_password,
                database=db_name
            )
            return "Hello World! Connected to the database successfully."
        except Exception as e:
            return f"Hello World! Failed to connect to database: {e}"
    
    if __name__ == "__main__":
        app.run(host="0.0.0.0", port=5000)

## 3. Python Requirements (app/requirements.txt)

    flask
    pymysql

## 4. Dockerfile

    # Use official Python image
    FROM python:3.9-slim
    
    # Set working directory
    WORKDIR /app
    
    # Copy application files
    COPY app/ /app/
    
    # Install dependencies
    RUN pip install --no-cache-dir -r requirements.txt
    
    # Expose the application port
    EXPOSE 5000
    
    # Run the application
    CMD ["python", "app.py"]

## 5. Docker Compose Template (docker-compose.yml)

    version: "3.8"
    
    services:
      app:
        build:
          context: .
          dockerfile: Dockerfile
        ports:
          - "5000:5000"
        env_file:
          - .env
        environment:
          - DB_HOST=${DB_HOST}
          - DB_USER=${DB_USER}
          - DB_PASSWORD=${DB_PASSWORD}
          - DB_NAME=${DB_NAME}

## 6. Environment Variables File (.env)

    DB_HOST=my-rds-instance.c2wkv3fxx123.us-east-1.rds.amazonaws.com
    DB_USER=admin
    DB_PASSWORD=yourpassword
    DB_NAME=mydatabase

## 7. Running the Application

    1. Build and Start the Containers

        **docker-compose up --build**

    2. Access the Application
      
      - Open your browser and navigate to http://localhost:5000.  IF you are running compose on your local system, if your are running on ec2 install, then you should first open port on SG(security group) and use instance public ip and port for accesing the application from internet like: "http://ip:port"
      - If the AWS RDS connection is successful, you will see a message:
      - Hello World! Connected to the database successfully.

## 8. AWS RDS Connection Steps

   1- To connect the containerized application with AWS RDS:
    
      Ensure RDS Instance is Accessible:
      
      **Make sure the RDS instance is publicly accessible(agr apna outside the vpc local system/instance sa RDS ko access krna ha) or resides in the same VPC as the container's host (if using EC2).
      Update the RDS security group to allow inbound traffic on port 3306/5432 from your container's IP or security group. **jb ap RDS create krty hn usi time ap apny created instance or rds ki connectivity ker sakhty hn, RDS khud hi 2 security group create kerta ha 1 for RDS or instance ki lye port open ker dye ga. (jis ma RDS SG ka inbound rule ma allow port db:3306/5432 ki port hogi or source ma instance k security group hoga. mean k instance rds ki 3306/5432 port per hit krna chahta ha tky traffic RDS ma jar sakhty so RDS apny SG ma instance ki traffic k lye port ko open kery or y kam sirf instance sa hi ho is k lye hum RDS ka SG ma inbound rule ma source instance k SG dety hn... tky RDS SG jb port open kry tu wo sirf instance k lye kery.., same port instance k outbound rule ma b add hogi. mean instance ki traffic ko instance k SG sa bahir nikalny k lye.. **
     
  2- Use .env Variables:
    
      The application reads AWS RDS credentials and endpoint from the .env file.
    
  3- Networking:
    
      If running containers locally, ensure your local machine can connect to the RDS instance.
      If running on AWS EC2, ensure the EC2 instance and RDS are in the same network and the security group rules are correct.
    
  4- Test the Connection:
    
      Use application logs or directly test the RDS connection using a tool like mysql or pymysql to verify connectivity.
      


