To utilize Foundry's AIP (Artificial Intelligence Platform) native Large Language Models (LLMs), you can integrate them into your workflows using AIP Logic, AIP Threads, and AIP Agents. Here's a step-by-step guide to get you started:

**1. Accessing AIP Logic:**

AIP Logic allows you to create functions that interact with your data using LLMs.

- **Navigate to AIP Logic:** In Foundry, go to the AIP Logic application.

- **Create a New Function:** Click on "Create Function" to start a new Logic function.

- **Configure Blocks:** Add blocks such as "Get object property," "Create variable," "Use LLM," and "Transform block" to define your function's workflow. citeturn0search12

**2. Implementing LLMs in AIP Logic:**

To integrate an LLM into your Logic function:

- **Add a "Use LLM" Block:** In your function, include a "Use LLM" block where you want the LLM to process data.

- **Configure the Block:** Set parameters like the LLM model to use, the prompt, and other settings.

- **Connect Blocks:** Link the "Use LLM" block to other blocks to define the data flow.

**3. Registering Custom LLMs:**

If you have a custom LLM, you can register it using function interfaces:

- **Set Up a REST Source:** Create a REST API source in Foundry to define your LLM's API endpoint.

- **Implement the ChatCompletion Interface:** Write a TypeScript function that calls your LLM via the REST source and implements the ChatCompletion function interface. citeturn0search1

- **Publish the Function:** Once implemented, publish the function to make it available for use in Foundry.

**4. Utilizing AIP Threads:**

AIP Threads enables interaction with documents using LLMs:

- **Upload Documents:** In AIP Threads, upload the documents you want to interact with.

- **Start a New Thread:** Create a new thread and select the documents to include.

- **Interact with the LLM:** Ask questions or provide prompts, and the LLM will process the documents to generate responses. citeturn0search6

**5. Creating AIP Agents:**

AIP Agents allow for more advanced interactions:

- **Access AIP Agent Studio:** Navigate to AIP Agent Studio in Foundry.

- **Define Agent Configuration:** Set up the agent's behavior, including the LLM model, prompts, and data sources.

- **Deploy the Agent:** Publish the agent to make it available for use in applications. citeturn0search14

**6. Integrating with Workshop:**

To display LLM-generated content in applications:

- **Add AIP Generated Content Widget:** In Workshop, include the AIP Generated Content widget on your page.

- **Configure the Widget:** Set it to display responses from your Logic function or directly from the LLM.

- **Customize Settings:** Adjust parameters like prompt variables and temperature to control the LLM's output. citeturn0search16

**Example Code Snippet for AIP Logic:**

Here's an example of how to configure a "Use LLM" block in AIP Logic:

```json
{
  "type": "Use LLM",
  "model": "gpt-4",
  "prompt": "Summarize the following text:",
  "temperature": 0.7
}
```

This configuration sets up the LLM to summarize the provided text with a temperature of 0.7, balancing creativity and coherence.

For more detailed tutorials and examples, refer to Palantir's official documentation and learning resources. citeturn0search4


To develop a Retrieval-Augmented Generation (RAG) application using Foundry's AIP native Large Language Models (LLMs), follow these steps:

**1. Set Up Your Environment:**

- **Create an AI Project:** In Foundry, initiate a new AI project to organize your resources.

- **Deploy AI Models:** Deploy the necessary LLMs, such as `gpt-4o-mini` for chat and `text-embedding-ada-002` for embeddings, to your project.

- **Add Azure AI Search:** Integrate Azure AI Search to index and retrieve relevant documents.

**2. Prepare Your Data:**

- **Add Application Data:** Incorporate your domain-specific data into Foundry.

- **Create a Search Index:** Utilize Azure AI Search to create an index of your data, enabling efficient retrieval.

**3. Develop the RAG Workflow:**

- **Retrieve Related Knowledge:** Use the search index to fetch documents pertinent to user queries.

- **Design Grounded Prompts:** Combine the retrieved documents with user inputs to craft prompts for the LLM.

- **Generate Responses:** Feed the grounded prompts into the LLM to produce contextually relevant answers.

**4. Implement the Application:**

- **Set Up Local Environment:** Configure your local development environment with necessary tools and libraries.

- **Develop Application Logic:** Write the code to handle user inputs, retrieve relevant documents, and generate responses using the LLM.

- **Test and Refine:** Iteratively test the application, refining prompts and retrieval methods to enhance performance.

For a comprehensive walkthrough, refer to the [RAG Chat App Tutorial on Azure AI Foundry](https://nitya.github.io/azure-ai-rag-workshop/).

Additionally, you might find the following video helpful:

videoBuilding a RAG workflow with AIP Logicturn0search8 
