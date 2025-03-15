# OpenSearch & OpenSearch Dashboards Setup on macOS (Apple Silicon)

This project demonstrates how to deploy a single-node OpenSearch instance with OpenSearch Dashboards using Docker Compose on macOS (Apple Silicon). Follow the steps below to configure your environment, initialize security, and access the dashboards.

## Prerequisites

-   **Docker Desktop**
    Ensure Docker Desktop is installed and configured with at least 4 GB of memory.
-   **Basic Command Line Knowledge**
    Familiarity with the terminal and Docker Compose is required.

## Environment Configuration

1.  **Create a `.env` File**

    In the project root, create a `.env` file to set your OpenSearch admin password.
    **Important:** Use `$$` for any literal dollar sign so that Docker Compose does not interpret it as an environment variable reference.

    Example structure:

    ```env
    # .env
    OPENSEARCH_INITIAL_ADMIN_PASSWORD=YOUR_STRONG_PASSWORD_HERE
    ```

    Replace `YOUR_STRONG_PASSWORD_HERE` with your actual secure password that meets OpenSearch’s requirements (minimum 8 characters, with uppercase, lowercase, digits, and special characters).

2.  **Docker Compose File**

    The project repository includes a preconfigured `docker-compose.yml` that defines both the OpenSearch and OpenSearch Dashboards services. Review it if needed, but no changes are required to follow these instructions.

## Starting the Services

1.  **Launch the Containers**

    Open your terminal in the project directory and run:

    ```bash
    docker compose up -d --force-recreate
    ```

    This command will download any necessary images and start the containers in detached mode.

2.  **Initialize OpenSearch Security**

    After the OpenSearch container starts, you might see a message like:

    ```nginx
    OpenSearch Security not initialized.
    ```

    To initialize the security configuration, run the following command:

    ```bash
    docker exec -it opensearch \
        /usr/share/opensearch/plugins/opensearch-security/tools/securityadmin.sh \
        -cd /usr/share/opensearch/config/opensearch-security/ \
        -icl -nhnv
    ```

    The flags used:

    -   `-cd`: Specifies the configuration directory.
    -   `-icl` and `-nhnv`: Instruct the script to ignore cluster and hostname verification (suitable for demo setups).

3.  **Verify OpenSearch**

    Test that OpenSearch is running by executing:

    ```bash
    curl -k -u 'admin:YOUR_STRONG_PASSWORD_HERE' https://localhost:9200
    ```

    You should receive a JSON response with cluster information.

## Accessing OpenSearch Dashboards

1.  **Open Your Browser**

    Navigate to `http://localhost:5601`.

2.  **Log In**

    Use the following credentials:

    -   Username: `admin`
    -   Password: The value set in your `.env` file (`YOUR_STRONG_PASSWORD_HERE`).

    If you encounter SSL certificate warnings (due to self-signed certificates in demo setups), you can proceed by bypassing them or adjusting the Dashboards configuration to disable strict SSL verification.

## Troubleshooting

-   **Environment Variable Issues:**

    If you see warnings about a variable (for example, a part of your password) not being set, double-check your `.env` file to ensure that all `$` characters are escaped (i.e., using `$$`).

-   **Container Connectivity:**

    Verify that your containers are running by checking:

    ```bash
    docker compose ps
    ```

    Also, ensure that no other application is using port 9200:

    ```bash
    lsof -i :9200
    ```

-   **Security Initialization:**

    If OpenSearch still reports that security is not initialized, re-run the security admin script and review the container logs:

    ```bash
    docker compose logs opensearch
    ```

-   **Dashboards Login Issues:**

    If you can access OpenSearch via `curl` but not via Dashboards:

    -   Ensure the Dashboards service environment variables (e.g., `OPENSEARCH_HOSTS`, `OPENSEARCH_USERNAME`, `OPENSEARCH_PASSWORD`, and `OPENSEARCH_SSL_VERIFICATIONMODE`) are set correctly.
    -   Check the Dashboards container logs with:

        ```bash
        docker compose logs opensearch-dashboards
        ```

    -   Try clearing your browser cache or using an incognito window.

## Conclusion

By following these steps, you should now have a fully operational single-node OpenSearch cluster along with OpenSearch Dashboards on your Mac (Apple Silicon) using Docker Compose. Adjust the settings as needed for development or testing purposes.