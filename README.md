# Spring Petclinic

## Running the Project with Docker

This project is configured to run in Docker containers.

### Prerequisites

*   Docker
*   Docker Compose

### Steps

1.  **Build and run the application:**
    Open a terminal in the and run the following commands:

    ```sh 
    cd PetClinic

    sudo docker compose up --build -d
    ```
2.  **Access the application:**
    Once the containers are up and running, you can access the Petclinic application by navigating to `http://localhost:8080` in your web browser.

    **Note:** There may be a short delay of 10-20 seconds after the command completes before the application is fully started and available in your browser. If you get a "connection refused" error, please wait a moment and try again.
