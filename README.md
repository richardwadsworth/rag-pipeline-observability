# OpenWebUI RAG System Setup Guide

This guide walks you through setting up a comprehensive Retrieval-Augmented Generation (RAG) system using OpenWebUI, Ollama, Langfuse, RabbitMQ, and n8n. When completed, you'll have a fully functional environment for creating and querying knowledge bases with custom LLM models, complete with observability and automated document processing workflows.

## ⚠️ Important Production Considerations

> **CAUTION**: The configuration described in this guide is intended for development and demonstration purposes only. It is **NOT** suitable for production environments or systems handling sensitive information.

Before deploying this system in a production environment, you should implement proper security measures including:

- Strong authentication mechanisms for all services
- Encrypted communication between components
- Secure storage of API keys and credentials
- Regular security updates and patches
- Proper network isolation and firewall rules
- Data backup and recovery procedures

Please refer to each component's official documentation for production-ready configuration guidelines:
- [OpenWebUI Documentation](https://docs.openwebui.com/)
- [Langfuse Documentation](https://langfuse.com/docs/)
- [RabbitMQ Documentation](https://www.rabbitmq.com/docs)
- [n8n Documentation](https://docs.n8n.io/)

## Example Knowledge Base Theme

This guide uses Harry Potter as the theme for our example knowledge base. The sample data consists of RSS feed outputs containing Harry Potter news and information. This theme was chosen to demonstrate:

1. How the system ingests and processes documents
2. The effectiveness of the vector database in storing and retrieving contextual information
3. How a custom-tuned model can leverage domain-specific knowledge to provide accurate responses
4. The complete pipeline from document ingestion to intelligent query responses

Throughout the guide, you'll see examples of queries related to Harry Potter that showcase the RAG system's capabilities.

## LLM Observability with Langfuse

Observability is a critical component of any production LLM system. This setup includes Langfuse, which provides comprehensive monitoring and analytics for your LLM applications. Key benefits include:

1. **Performance Monitoring**: Track response times, token usage, and model performance metrics to optimise your system.
2. **Cost Tracking**: Monitor token consumption and associated costs to manage your budget effectively.
3. **Quality Assurance**: Evaluate response quality and identify patterns in user interactions that might require model adjustments.
4. **Debugging**: Trace the complete lifecycle of requests through your system to identify bottlenecks or errors.
5. **Continuous Improvement**: Use insights from real usage patterns to refine prompts, knowledge bases, and model selection.

The integration between OpenWebUI and Langfuse in this setup allows you to visualise these metrics through an intuitive dashboard, helping you build more reliable and effective LLM applications.

## Table of Contents

- [OpenWebUI RAG System Setup Guide](#openwebui-rag-system-setup-guide)
  - [⚠️ Important Production Considerations](#️-important-production-considerations)
  - [Example Knowledge Base Theme](#example-knowledge-base-theme)
  - [LLM Observability with Langfuse](#llm-observability-with-langfuse)
  - [Table of Contents](#table-of-contents)
  - [Prerequisites](#prerequisites)
  - [System Architecture](#system-architecture)
  - [Initial Setup](#initial-setup)
  - [Ollama Setup (Optional)](#ollama-setup-optional)
  - [OpenWebUI Setup](#openwebui-setup)
    - [Initial Configuration](#initial-configuration)
    - [Model Management](#model-management)
    - [Knowledge Base Creation](#knowledge-base-creation)
    - [Custom Model Creation](#custom-model-creation)
    - [API Key Generation](#api-key-generation)
  - [Langfuse Setup](#langfuse-setup)
    - [Langfuse Initial Configuration](#langfuse-initial-configuration)
    - [Integration Langfuse with OpenWebUI](#integration-langfuse-with-openwebui)
  - [RabbitMQ Setup](#rabbitmq-setup)
  - [n8n Workflow Setup](#n8n-workflow-setup)
    - [Workflow Configuration](#workflow-configuration)
    - [Testing the Workflow](#testing-the-workflow)
  - [Testing the Complete System](#testing-the-complete-system)
  - [Troubleshooting](#troubleshooting)
  - [Additional Notes](#additional-notes)
    - [Managing Documents in Knowledge Base](#managing-documents-in-knowledge-base)
    - [RabbitMQ's Role in Document Processing](#rabbitmqs-role-in-document-processing)

## Prerequisites

Before beginning, ensure you have the following installed on your Linux VM:

- Docker (20.10.x or newer)
- Docker Compose (2.x or newer)
- Git (optional, for cloning repositories)
- At least 16GB RAM for running multiple containers
- At least 50GB free disk space for models and data if also running the optional ollama container

## System Architecture

This setup includes the following components:

- **Ollama**: For running local LLM models (optional)
- **OpenWebUI**: Web interface for interacting with LLMs and managing knowledge bases
- **Langfuse**: For LLM observability and analytics
- **RabbitMQ**: Message broker for communication between services
- **n8n**: Workflow automation for processing documents into knowledge bases

## Initial Setup

Create the necessary directory structure in the VM:

```bash
# Create main directory
mkdir -p ~/docker
cd ~/docker
```

## Ollama Setup (Optional)

This section covers setting up Ollama, which provides local LLM inference capabilities. You can skip this step if you already have an Ollama instance running elsewhere that you can connect to. If you're setting up a complete local environment, follow these steps to deploy Ollama in a Docker container.

Set up the Ollama container:

```bash
# Create Ollama directory
mkdir -p ollama
cd ollama

# Create docker-compose.yml file
nano docker-compose.yml
# Copy the Ollama docker-compose.yml content here

# Start Ollama container
docker compose up -d

# Return to main directory
cd ..
```

## OpenWebUI Setup

Set up the OpenWebUI container:

```bash
# Create OpenWebUI directory
mkdir -p openwebui
cd openwebui

# Create required subdirectories
mkdir -p data/pipeline-input

# Create docker-compose.yml file
nano docker-compose.yml
# Copy the OpenWebUI docker-compose.yml content here

### Environment Configuration

Before starting the OpenWebUI container, you need to configure the environment variables:

1. Create an `.env` file in the same directory as your `docker-compose.yml`:

   ```bash
   nano .env
   ```

2. Add the following environment variables to the `.env` file:

   ```bash
   # Ollama Configuration
   OLLAMA_BASE_URL=host.docker.internal
   ```

3. If you're using a remote Ollama instance, update the `OLLAMA_BASE_URL` variable with the IP address or hostname of your remote Ollama server.

4. Save the file and exit the editor.

5. Start the OpenWebUI container:

   ```bash
   docker compose up -d
   ```

6. Log all configuration changes to the logs directory:

   ```bash
   mkdir -p logs
   echo "Environment configuration completed at $(date)" | tee -a logs/setup.log
   ```

### Initial Configuration

The following steps will guide you through the initial setup of OpenWebUI, including creating an admin account and configuring the basic settings. This will establish the foundation for model management, knowledge base creation, and custom model configuration.

1. Access the OpenWebUI interface at <http://<vm-ip-address>:3000/>

2. Sign up with the following credentials:
   - Email: admin@domain.com
   - Password: Password101!

### Model Management

1. Navigate to Admin Panel → Settings → Models
2. Click on "Manage models"
3. To pull a model from Ollama:
   - Enter a model tag (e.g., `llama3.1:8b`)
   - Click the download button
   - Be patient as the model downloads (there is no progress bar)

### Knowledge Base Creation

1. Navigate to Workspace → Knowledge
2. Click the plus (+) symbol to add a new Knowledge Base
3. Configure the Knowledge Base:
   - Name: (e.g., PotterKB001)
   - Description: Add a relevant description
   - Visibility: Public
4. Click "Create Knowledge"
5. **Important**: Note the GUID in the URL, as this is the Knowledge Base ID
   - Example: <http://localhost:3000/workspace/knowledge/7045719c-3581-4c55-9188-27e4ded9d524>
   - The ID in this example is `7045719c-3581-4c55-9188-27e4ded9d524`

### Custom Model Creation

1. Navigate to Workspace → Model
2. Create a new model:
   - Name: (e.g., Potter1.0)
   - Select a base model from the dropdown list
   - System prompt: "You are a Harry Potter analyst and expert in all things Harry Potter. Answer the questions using the context provided."
3. Save the model configuration

### API Key Generation

1. Navigate to Settings → Account → API Keys
2. Click "Show" then "New API Key"
3. Copy the generated API key (this will be your bearer token)
4. Click "Save"

## Langfuse Setup

Langfuse provides observability for your LLM applications, allowing you to track, monitor, and analyse model performance. The following steps will help you set up Langfuse and integrate it with OpenWebUI to gain insights into your model's behaviour and responses.

### Langfuse Initial Configuration

1. Access Langfuse at <http://<vm-ip-address>:3001/>
2. Sign up with the following credentials:
   - Email: admin@domain.com
   - Password: Password101!
3. Create a new organisation named "my-org"
4. Create a new project named "my-llm-project"
5. Click "Create new API Key" button
6. Note the secret and private key credentials in the project settings

### Integration Langfuse with OpenWebUI

1. Navigate to OpenWebUI at <http://<vm-ip-address>:3000/>
2. Go to Admin Panel → Settings → Pipelines
3. Upload the Langfuse filter pipeline file from `/pipelines/langfuse_filter_pipeline.py`
4. Configure the pipeline:
   - Add the Langfuse Secret Key
   - Add the Langfuse Private Key
   - Update the hostname to <http://host.docker.internal:3001/>

To test the Langfuse integration:

1. Start a new chat in OpenWebUI
2. Enter any question such as "how are you?"
3. Check Langfuse at <http://<vm-ip-address>:3001/>
4. Navigate to "Traces" to see the new trace entry for you question to the llm

## RabbitMQ Setup

RabbitMQ serves as the message broker for the system, enabling asynchronous communication between components. This setup creates a queue that will be used by n8n to process documents and add them to your knowledge base.

1. Access the RabbitMQ management interface at <http://<vm-ip-address>:15672/>
2. Sign in with default credentials:
   - Username: guest
   - Password: guest
3. Create a new queue:
   - Navigate to Queues → Add a new Queue
   - Queue name: openwebui
   - Click "Add Queue"

## n8n Workflow Setup

n8n provides the automation capabilities for your RAG system, allowing you to create workflows that process documents and add them to your knowledge base. The following steps will help you set up and configure n8n to automatically process files placed in a designated input directory.

1. Access n8n at <http://<vm-ip-address>:5678/>
2. Sign up with the following credentials:
   - Email: admin@domain.com
   - Password: Password101!

### Workflow Configuration

1. Import the workflow:
   - Click "Create Workflow"
   - Click the 3 dots in the top right
   - Select "Import from File"
   - Browse to `/n8n/AddFileToOpenWebUiKB.json`
   - Click "Add"
   - Click "Save"

2. Configure the "Post File" step:
   - Click on the "Post File" node
   - Update the Headers Parameter with your OpenWebUI bearer token you created earlier:
     - Format: `Bearer sk-38ebf3557fa1417455f83853a3700b83`
   - Click "Back to Canvas"

3. Configure the "Add File To KB" step:
   - Click on the "Add File To KB" node
   - Update the URL with your Knowledge Base ID:
     - Format: <http://host.docker.internal:3000/api/v1/knowledge/7045719c-3581-4c55-9188-27e4ded9d524/file/add>
   - Update the Headers Parameter with your OpenWebUI bearer token
   - Click "Back to Canvas"

4. Configure RabbitMQ connections:
   - Click on the "Rabbit MQ" node
   - Click "Select Credential Dropdown" → "Create new Credential"
   - Configure:
     - Hostname: host.docker.internal
     - Leave other settings as default
   - Click "Save"
   - Verify "Connection tested successfully" message
   - Repeat for the "Rabbit MQ Trigger" node

### Testing the Workflow

1. Activate the workflow by changing its state from "Inactive" to "Active"
2. Add a test file to the input directory:
   - In the VM, navigate to `~/docker/openwebui/data/pipeline-input`
   - Copy the sample file `/data/rss/2025-01-02T2122_Re__Google_Alert_-_harry_potter.html` to this directory
3. Check workflow execution:
   - Click on the "Executions" button in n8n
   - Verify both events:
     - Local File Trigger firing
     - RabbitMQ trigger firing
4. Verify the file was added to the Knowledge Base:
   - Browse to your OpenWebUI Knowledge Base (<http://<vm-ip-address>:3000/workspace/knowledge/<your-kb-id>>)
   - Confirm the file appears in the Knowledge Base

## Testing the Complete System

1. Navigate to OpenWebUI at <http://<vm-ip-address>:3000/>
2. Start a new chat
3. Select your custom model (e.g., Potter1.0) from the models dropdown
4. Ask a question related to the content in your Knowledge Base, for example "which school became hogwarts for the day?"
5. Verify that the model provides an answer using information from your Knowledge Base

## Troubleshooting

- **Container not starting**: Check logs with `docker logs <container_name>`
- **Network issues**: Ensure all containers can communicate using `host.docker.internal`
- **Model download failures**: Check disk space and network connectivity
- **n8n workflow errors**: Check credentials and API endpoints in each node
- **Knowledge Base not updating**: Verify file permissions and webhook configurations

For additional help, check the logs in each container or refer to the respective documentation.

## Additional Notes

### Managing Documents in Knowledge Base

OpenWebUI does not currently support selectively removing individual documents from the underlying Chroma vector database. If you need to delete a file from the Knowledge Base, the recommended approach is to:

1. Reset the vector database
2. Recreate the Knowledge Base and custom model
3. Update the n8n workflow with the new Knowledge Base ID

To reset the vector database:
1. Navigate to OpenWebUI at <http://<vm-ip-address>:3000/admin/settings>
2. Scroll down to the "Reset Vector Storage/Knowledge" section
3. Click the "Reset" button
4. Confirm the action when prompted

### RabbitMQ's Role in Document Processing

RabbitMQ is a critical component in this solution because the OpenWebUI API cannot handle parallel indexing of documents. All documents must be processed serially to ensure proper indexing. Without RabbitMQ:

- Copying multiple documents to the workflow's `pipeline-input` directory would trigger simultaneous processing
- Parallel processing would cause indexing failures due to API limitations
- Document content might not be properly stored in the vector database

RabbitMQ ensures that documents are queued and processed one at a time, maintaining the integrity of the indexing process and preventing API overload.
