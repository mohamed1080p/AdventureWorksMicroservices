# AdventureWorksMicroservices


## Project Objectives

The goal of this project is to implement a microservices-based application for managing AdventureWorks data, including customers, orders, and inventory. This project leverages modern technologies such as Docker for containerization and Jenkins for continuous integration and deployment (CI/CD).

## Technology Stack

- **Backend:** ASP.NET Core Web API
- **Database:** MS SQL Server
- **Containerization:** Docker, Docker Compose
- **CI/CD:** Jenkins
- **Version Control:** GitHub
- **ORM:** Entity Framework Core
- **Documentation:** Swagger

## Setup Instructions



### Prerequisites
Before you begin, ensure you have the following installed on your machine:
- [Git](https://git-scm.com/downloads)
- [.NET Core SDK](https://dotnet.microsoft.com/download)
- [MS SQL Server](https://www.microsoft.com/en-us/sql-server/sql-server-downloads)
- [Docker Desktop](https://www.docker.com/products/docker-desktop)
- [Jenkins](https://www.jenkins.io/download/) (optional for CI/CD setup)

### Step 1: Clone the Repository
1. Open your terminal or command prompt.
2. Clone the repository to your local machine:
   ```bash
   git clone https://github.com/yourusername/AdventureWorksMicroservices.git

3.Navigate into the cloned directory:
cd AdventureWorksMicroservices

Step 2: Set Up Version Control
1.Initialize Git (if not already initialized):
git init

2.Create a .gitignore file to exclude unnecessary files:
echo "bin/
obj/
*.log
*.db" > .gitignore

3.Add and commit the initial files:
git add .
git commit -m "Initialize project with README and .gitignore"

Step 3: Set Up MS SQL Server and AdventureWorks Database
1.Install MS SQL Server:
-Download and install MS SQL Server on your machine.

2-Download AdventureWorks Database:
-Download the AdventureWorks sample database from Microsoft.

3.Attach the Database:
-Open SQL Server Management Studio (SSMS).
-Connect to your SQL Server instance.
-Right-click on Databases and select Attach.
-Browse and select the downloaded AdventureWorks.mdf file.
-Click OK to attach the database.


Step 4: Configure the REST API Project
1.Open the Project in Visual Studio:
-Navigate to the project directory and open the .sln file in Visual Studio.

2.Configure Connection String:
-Open appsettings.json and update the connection string:

"ConnectionStrings": {
  "DefaultConnection": "Server=localhost;Database=AdventureWorks;User Id=your_username;Password=your_password;"
}

3.Restore Dependencies and Run the Application:
(bash)
dotnet restore
dotnet run

The API should now be running at http://localhost:5000 (or the port specified in your launch settings).


5.Set Up Docker Containers:
1.Create Dockerfile for REST API:
-In the project folder, create a file named Dockerfile with the following content:

FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
COPY . .
ENTRYPOINT ["dotnet", "YourApiProjectName.dll"]

2.Pull MS SQL Server Docker Image:

docker pull mcr.microsoft.com/mssql/server

3.Create docker-compose.yml:
-In the root directory, create a docker-compose.yml file:

version: '3.8'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server
    environment:
      SA_PASSWORD: "YourStrong@Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
  api:
    build: .
    depends_on:
      - sqlserver
    environment:
      ConnectionStrings__DefaultConnection: "Server=sqlserver;Database=AdventureWorks;User Id=sa;Password=YourStrong@Passw0rd;"
    ports:
      - "8080:80"

4.Build and Run Containers:
docker-compose up --build
-The REST API will be accessible at http://localhost:8080.

Step 6: Set Up CI/CD Pipeline with Jenkins (Optional)
1.Install Jenkins:

-Download and install Jenkins from here.
-Start Jenkins and access it at http://localhost:8080.

2.Configure GitHub Integration:
-In your GitHub repository, navigate to Settings > Webhooks.
-Add a new webhook with the Jenkins URL (e.g., http://localhost:8080/github-webhook/) to trigger builds on commits.

3.Create Jenkins Pipeline:
-Add a Jenkinsfile to your repository with the following content:

pipeline {
  agent any
  stages {
    stage('Build') {
      steps {
        sh 'dotnet build AdventureWorksMicroservices.sln'
      }
    }
    stage('Docker Build') {
      steps {
        sh 'docker build -t adventureworks-api .'
      }
    }
    stage('Deploy to Staging') {
      steps {
        sh 'docker-compose up --build -d'
      }
    }
  }
}

Architecture Overview
Project Structure
1.API Service: ASP.NET Core Web API for CRUD operations.
2.Database Service: MS SQL Server hosting the AdventureWorks database.
3.Containerization: Docker for application and database.
4.CI/CD Pipeline: Jenkins automates build and deployment.




-------------------------------------------------------------------------------------------------------------------
#!/bin/bash

# AdventureWorks Microservices Setup Script

# Color codes for better readability
GREEN='\033[0;32m'
YELLOW='\033[1;33m'
RED='\033[0;31m'
NC='\033[0m' # No Color

# Function to print status messages
print_status() {
    echo -e "${GREEN}[✓] $1${NC}"
}

print_warning() {
    echo -e "${YELLOW}[!] $1${NC}"
}

print_error() {
    echo -e "${RED}[✗] $1${NC}"
}

# Check Prerequisites
check_prerequisites() {
    print_status "Checking Prerequisites..."
    
    # Check Git
    if ! command -v git &> /dev/null; then
        print_error "Git is not installed"
        exit 1
    fi

    # Check .NET Core SDK
    if ! command -v dotnet &> /dev/null; then
        print_error ".NET Core SDK is not installed"
        exit 1
    fi

    # Check Docker
    if ! command -v docker &> /dev/null; then
        print_error "Docker is not installed"
        exit 1
    fi

    print_status "All prerequisites are met!"
}

# Clone Repository
clone_repository() {
    print_status "Cloning Repository..."
    
    # Prompt for GitHub username
    read -p "Enter your GitHub username: " USERNAME
    
    git clone https://github.com/$USERNAME/AdventureWorksMicroservices.git
    cd AdventureWorksMicroservices
    
    print_status "Repository cloned successfully!"
}

# Setup Version Control
setup_version_control() {
    print_status "Setting up Version Control..."
    
    # Initialize Git
    git init
    
    # Create .gitignore
    echo "bin/
obj/
*.log
*.db" > .gitignore
    
    # Add and commit initial files
    git add .
    git commit -m "Initialize project with README and .gitignore"
    
    print_status "Version control setup complete!"
}

# Configure Connection String
configure_connection_string() {
    print_status "Configuring Connection String..."
    
    # Prompt for database credentials
    read -p "Enter SQL Server username: " DB_USER
    read -sp "Enter SQL Server password: " DB_PASSWORD
    echo
    
    # Update appsettings.json (assumes file exists)
    sed -i "s|\"DefaultConnection\": \".*\"|\"DefaultConnection\": \"Server=localhost;Database=AdventureWorks;User Id=$DB_USER;Password=$DB_PASSWORD;\"|g" appsettings.json
    
    print_status "Connection string updated!"
}

# Build and Run Application
build_and_run_application() {
    print_status "Building and Restoring Application..."
    
    dotnet restore
    dotnet build
    
    print_warning "Start the application? (y/n)"
    read START_APP
    if [[ $START_APP == "y" ]]; then
        dotnet run
    fi
}

# Create Docker Files
create_docker_files() {
    print_status "Creating Docker Configuration..."
    
    # Create Dockerfile
    cat > Dockerfile << EOL
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 80
COPY . .
ENTRYPOINT ["dotnet", "AdventureWorksMicroservices.dll"]
EOL

    # Create docker-compose.yml
    cat > docker-compose.yml << EOL
version: '3.8'
services:
  sqlserver:
    image: mcr.microsoft.com/mssql/server
    environment:
      SA_PASSWORD: "YourStrong@Passw0rd"
      ACCEPT_EULA: "Y"
    ports:
      - "1433:1433"
  
  api:
    build: .
    depends_on:
      - sqlserver
    environment:
      ConnectionStrings__DefaultConnection: "Server=sqlserver;Database=AdventureWorks;User Id=sa;Password=YourStrong@Passw0rd;"
    ports:
      - "8080:80"
EOL

    print_status "Docker configuration files created!"
}

# Build and Run Docker Containers
run_docker_containers() {
    print_warning "Build and run Docker containers? (y/n)"
    read RUN_DOCKER
    
    if [[ $RUN_DOCKER == "y" ]]; then
        # Pull MS SQL Server Docker image
        docker pull mcr.microsoft.com/mssql/server
        
        # Build and run containers
        docker-compose up --build
    fi
}

# Optional Jenkins Pipeline Configuration
setup_jenkins_pipeline() {
    print_warning "Set up Jenkins pipeline? (y/n)"
    read SETUP_JENKINS
    
    if [[ $SETUP_JENKINS == "y" ]]; then
        # Create Jenkinsfile
        cat > Jenkinsfile << EOL
pipeline {
    agent any
    stages {
        stage('Build') {
            steps {
                sh 'dotnet build AdventureWorksMicroservices.sln'
            }
        }
        stage('Docker Build') {
            steps {
                sh 'docker build -t adventureworks-api .'
            }
        }
        stage('Deploy to Staging') {
            steps {
                sh 'docker-compose up --build -d'
            }
        }
    }
}
EOL
        
        print_status "Jenkinsfile created! Configure in your Jenkins instance."
    fi
}

# Main Script Execution
main() {
    clear
    echo "AdventureWorks Microservices Setup Script"
    echo "========================================"
    
    check_prerequisites
    clone_repository
    setup_version_control
    configure_connection_string
    build_and_run_application
    create_docker_files
    run_docker_containers
    setup_jenkins_pipeline
    
    print_status "Setup Complete! Happy Coding!"
}

# Run the main script
main



















