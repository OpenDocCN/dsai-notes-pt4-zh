<!--yml
category: 未分类
date: 2025-01-11 11:57:08
-->

# Adaptive REST API Testing with Reinforcement Learning

> 来源：[https://arxiv.org/html/2411.07098/](https://arxiv.org/html/2411.07098/)

###### Abstract

Modern web services increasingly rely on REST APIs. Effectively testing these APIs poses challenges due to the vast search space to explore, which involves selecting API operations for sequence creation, choosing parameters for each operation from a potentially large set, and sampling values from the often infinite parameter input space. Current testing tools lack efficient exploration mechanisms, treating all operations and parameters equally without considering their importance or complexity and lacking prioritization strategies. Furthermore, these tools struggle when response schemas are absent in the specification or exhibit variants. To address these limitations, we present an adaptive REST API testing technique that incorporates reinforcement learning to prioritize operations and parameters for exploration. Our approach dynamically analyzes request and response data to inform dependent parameters and adopts a sampling-based strategy for efficient processing of dynamic API feedback. We evaluate our technique on ten RESTful services, comparing it against state-of-the-art tools with respect to code coverage achieved and the number of generated requests, operations covered, and service failures triggered. Additionally, we perform an ablation study on prioritization, dynamic feedback analysis, and sampling to assess their individual effects. Our findings demonstrate that our approach significantly outperforms existing REST API testing tools in terms of effectiveness, efficiency, and fault-finding ability.

## I Introduction

The increasing adoption of modern web services has led to a growing reliance on REpresentational State Transfer (REST) APIs for communication and data exchange [richardson2013restful, patni2017pro]. REST APIs adhere to a set of architectural principles that enable scalable, flexible, and efficient interactions between various software components through the use of standard HTTP methods and a stateless client-server model [fielding2000architectural]. To facilitate their discovery and use by clients, REST APIs are often documented using specification languages [openapi, swagger, raml, apiblueprint], which let developers describe the APIs in a structured format and provide essential information, such as the available endpoints, input parameters and their schemas, response schemas, etc. Platforms such as APIs Guru [apis_guru] host thousands of RESTful API documents, emphasizing the significance of these standardized API specifications in industry.

Standardized documentation formats, such as the OpenAPI specification [openapi], not only facilitate the development of APIs and their use by clients, but also provide a foundation for the development of automated testing techniques for REST APIs, and numerous such techniques and tools have emerged in recent years (e.g.,  [arcuri2019restful, Corradini2022, atlidakis2019restler, martin2020restest, karlsson2020automatic, karlsson2020quickrest, liu2022morest, wu2022combinatorial]). In spite of this, effectively testing REST APIs continues to be a challenge, with high code coverage remaining an elusive goal for tools [kim2022automated].

Testing REST APIs can be challenging because of the large search space for exploration, arising from numerous operations, potential execution orders, inter-parameter dependencies, and associated input parameter value constraints [martin2019catalogue]. Current techniques often struggle to explore this space due to the lack effective exploration strategies for operations and their parameters. Existing testing tools tend to treat all operations and parameters equally, disregarding their importance or complexity, leading to suboptimal testing strategies and insufficient coverage of crucial operation and parameter combinations. Moreover, these tools rely on discovering producer-consumer relationships between response schemas and request parameters, which works well when the parameter and response schemas are described in detail in the specification. However, if the schemas are incomplete or imprecise, the tools can become less effective in their exploration.

In this paper, we present adaptive REST API testing with reinforcement learning (arat-rl), an advanced black-box testing approach that addresses these limitations of existing tools. Our technique incorporates several innovative features, such as leveraging reinforcement learning to prioritize operations and parameters for exploration, dynamically constructing key-value pairs from both response and request data, analyzing these pairs to inform dependent operations and parameters, and utilizing a sampling-based strategy for efficient processing of dynamic API feedback. The primary objective of our approach is to increase code coverage and improve fault-detection capability.

The core of novelty in arat-rl is an adaptive testing strategy, driven by a reinforcement-learning-based prioritization algorithm for exploring the space of operations and parameters. The algorithm initially determines an operation’s importance based on the parameters used and their frequencies across other operations. This targeted exploration enables efficient coverage of critical operations and parameters, thereby optimizing code coverage. The technique employs reinforcement learning to adjust the priority weights associated with operations and parameters based on feedback, by decreasing importance for successful responses and increasing it for failed responses. The technique also assigns weights to parameter-value mappings based on various sources of input values (e.g., random, specified values, response values, request values, and default values), which lets it adapt the testing strategy and concentrate on areas more likely to contain faults, ultimately enhancing fault-detection capability.

Another innovative feature of arat-rl is dynamic construction of key-value pairs. In contrast to existing approaches that rely heavily on resource schemas provided in the specification, our technique dynamically constructs key-value pairs by analyzing POST operations (i.e., resource creation HTTP method) and examining both response and request data. For instance, support that an operation takes book title and price as request parameters and, as response, produces a success status code along with a string message (e.g., “Successfully created”). Our technique leverages this information to create key-value pairs for book title and price, upon receiving a successful response, even without such data being present in the response. It takes into account the input parameters used in the request, as they correspond to the created resource. Moreover, if the service returns incomplete resources, our technique still processes the information available in key-value pairs. This dynamic approach enables our tool to identify resources from the API responses and requests and discover hidden dependencies that are not evident from the specification alone.

Finally, arat-rl employs a simple yet effective sampling-based approach that allows it to process dynamic API feedback efficiently and adapt its exploration based on the gathered information. By randomly sampling key-value pairs from responses, our tool reduces the overhead of processing every response for each pair, resulting in more efficient testing and optimized utilization of testing resources.

To evaluate the technique, we conducted empirical studies using 10 RESTful services and compared it against three state-of-the-art REST API testing tools: RESTler [atlidakis2019restler], EvoMaster [arcuri2019restful], and Morest [liu2022morest]. We assessed the effectiveness of arat-rl in terms of coverage achieved and service failures triggered, and its efficiency in terms of valid and fault-inducing requests generated and operations covered within a given time budget. Our results show that arat-rl outperforms the competing tools in all the metrics considered—it achieved the highest method, branch, and line coverage rates, along with better fault-detection ability. Specifically, arat-rl covered 119.17%, 59.83%, and 52.42% more branches, lines, and methods than RESTler; 37.03%, 20.87%, and 14.13% more branches, lines, and methods than EvoMaster; and 23.69%, 11.87%, and 9.55% more branches, lines, and methods than Morest. arat-rl also uncovered 9.2x, 2.5x, and 2.4x more bugs than RESTler, EvoMaster, and Morest, respectively. In terms of efficiency, arat-rl generated 52.01%, 40.79%, and 1222% more valid and fault-inducing requests and covered 15.38%, 24.14%, and 282.98% more operations than Morest, EvoMaster, and RESTler, respectively, in a one-hour testing time budget. We also conducted an ablation study to assess the individual effects of prioritization, dynamic feedback analysis, and sampling on the overall effectiveness of arat-rl. Our results indicate that reinforcement-learning-based prioritization contributes the most to arat-rl’s effectiveness, followed by dynamic feedback analysis and sampling in that order.

[⬇](data:text/plain;base64,L3Byb2R1Y3RzL3twcm9kdWN0TmFtZX0vY29uZmlndXJhdGlvbnMve2NvbmZpZ3VyYXRpb25OYW1lfS9mZWF0dXJlcy97ZmVhdHVyZU5hbWV9OgogQFxjb2xvcntrZXl3b3JkY29sb3J9cG9zdEA6CiAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1vcGVyYXRpb25JZEA6IGFkZEZlYXR1cmVUb0NvbmZpZ3VyYXRpb24KICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXByb2R1Y2VzQDoKICAgICAgLSBhcHBsaWNhdGlvbi9qc29uCiAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1wYXJhbWV0ZXJzQDoKICAgICAgLSBAXGNvbG9ye2tleXdvcmRjb2xvcn1uYW1lQDogcHJvZHVjdE5hbWUKICAgICAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1pbkA6IHBhdGgKICAgICAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1yZXF1aXJlZEA6IHRydWUKICAgICAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn10eXBlQDogc3RyaW5nCiAgICAgIC0gQFxjb2xvcntrZXl3b3JkY29sb3J9bmFtZUA6IGNvbmZpZ3VyYXRpb25OYW1lCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9aW5AOiBwYXRoCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9cmVxdWlyZWRAOiB0cnVlCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9dHlwZUA6IHN0cmluZwogICAgICAtIEBcY29sb3J7a2V5d29yZGNvbG9yfW5hbWVAOiBmZWF0dXJlTmFtZQogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfWluQDogcGF0aAogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXJlcXVpcmVkQDogdHJ1ZQogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXR5cGVAOiBzdHJpbmcKICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXJlc3BvbnNlc0A6CiAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfWRlZmF1bHRAOgogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfWRlc2NyaXB0aW9uQDogc3VjY2Vzc2Z1bCBvcGVyYXRpb24KL3Byb2R1Y3RzL3twcm9kdWN0TmFtZX0vY29uZmlndXJhdGlvbnMve2NvbmZpZ3VyYXRpb25OYW1lfS9mZWF0dXJlczoKICBAXGNvbG9ye2tleXdvcmRjb2xvcn1nZXRAOgogICAgQFxjb2xvcntrZXl3b3JkY29sb3J9b3BlcmF0aW9uSWRAOiBnZXRDb25maWd1cmF0aW9uQWN0aXZlZEZlYXR1cmVzCiAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1wcm9kdWNlc0A6CiAgICAgIC0gYXBwbGljYXRpb24vanNvbgogICAgQFxjb2xvcntrZXl3b3JkY29sb3J9cGFyYW1ldGVyc0A6CiAgICAgIC0gQFxjb2xvcntrZXl3b3JkY29sb3J9bmFtZUA6IHByb2R1Y3ROYW1lCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9aW5AOiBwYXRoCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9cmVxdWlyZWRAOiB0cnVlCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9dHlwZUA6IHN0cmluZwogICAgICAtIEBcY29sb3J7a2V5d29yZGNvbG9yfW5hbWVAOiBjb25maWd1cmF0aW9uTmFtZQogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfWluQDogcGF0aAogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXJlcXVpcmVkQDogdHJ1ZQogICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXR5cGVAOiBzdHJpbmcKICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXJlc3BvbnNlc0A6CiAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfScyMDAnQDoKICAgICAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn1kZXNjcmlwdGlvbkA6IHN1Y2Nlc3NmdWwgb3BlcmF0aW9uCiAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9c2NoZW1hQDoKICAgICAgICAgIEBcY29sb3J7a2V5d29yZGNvbG9yfXR5cGVAOiBhcnJheQogICAgICAgICAgQFxjb2xvcntrZXl3b3JkY29sb3J9aXRlbXNAOgogICAgICAgICAgICBAXGNvbG9ye2tleXdvcmRjb2xvcn10eXBlQDogc3RyaW5n)/products/{productName}/configurations/{configurationName}/features/{featureName}:post:operationId:  addFeatureToConfigurationproduces:-  application/jsonparameters:-  name:  productNamein:  pathrequired:  truetype:  string-  name:  configurationNamein:  pathrequired:  truetype:  string-  name:  featureNamein:  pathrequired:  truetype:  stringresponses:default:description:  successful  operation/products/{productName}/configurations/{configurationName}/features:get:operationId:  getConfigurationActivedFeaturesproduces:-  application/jsonparameters:-  name:  productNamein:  pathrequired:  truetype:  string-  name:  configurationNamein:  pathrequired:  truetype:  stringresponses:’200’:description:  successful  operationschema:type:  arrayitems:type:  string

Figure 1: A Part of Features-Service’s OpenAPI Specification.

The main contributions of this work are:

*   •

    A novel approach for adaptive REST API testing that incorporates (1) reinforcement learning to prioritize exploration of operations and parameters, (2) dynamic analysis of request and response data to inform dependent parameters, and (3) a sampling-based strategy for efficient processing of dynamic API feedback.

*   •

    Empirical results demonstrating that arat-rl outperforms state-of-the-art REST API testing tools in terms of requests generated, code coverage achieved, and service failures triggered.

*   •

    An artifact [artifact] containing the tool, the benchmark services, and the empirical data.

## II Background and Motivating Example

We provide a brief introduction to REST APIs, the OpenAPI specification, and reinforcement learning, and then illustrate the novel features of our approach using a running example.

### II-A REST APIs

REST APIs are web APIs that adhere to the RESTful architectural style [fielding2000architectural]. REST APIs facilitate communication between clients and servers by exchanging data through standardized protocols, such as HTTP [rodriguez2008restful]. Clients communicate with web services by sending HTTP requests. These requests access and/or manipulate resources managed by the service, where a resource represents data that a client may want to create, delete, update, or access. Requests are sent to an API endpoint, identified by a resource path and an HTTP method specifying the action to be performed on the resource. The most commonly used methods are POST, GET, PUT, and DELETE, for creating, reading, updating, and deleting a resource, respectively. The combination of an endpoint and an HTTP method is called an operation. Besides specifying an operation, a request can also optionally include HTTP headers containing metadata and a body with the request’s payload. Upon receiving and processing a request, the web service returns a response containing headers, possibly a body, and an HTTP status code—a three-digit number indicating the request’s outcome.

### II-B OpenAPI Specification

The OpenAPI Specification (OAS) [openapi] is a widely adopted API description format for RESTful APIs, providing a standardized and human-readable way to describe the structure, functionality, and expected behavior of an API. Figure [1](https://arxiv.org/html/2411.07098v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Adaptive REST API Testing with Reinforcement Learning") illustrates an example OAS file describing a part of a Features-Service API. This example shows two API operations. The first operation, a POST request, is designed to add a feature name to a product’s configuration. It requires three parameters: the product name, configuration name, and feature name, all of which are specified in the path. Upon successful execution, the API responds with a JSON object, signaling that the feature has been added to the configuration. The second operation, a GET request, retrieves the active features of a product’s configuration. Similar to the first operation, it requires the product name and configuration name as path parameters. The API responds with a 200 status code and an array of strings representing the active features in the specified configuration.

### II-C Reinforcement Learning and Q-Learning

Reinforcement learning (RL) is a type of machine learning where an agent learns to make decisions by interacting with an environment [sutton2018reinforcement]. The agent selects actions in various situations (states), observes the consequences (rewards), and learns to choose the best actions to maximize the cumulative reward over time. The learning process in RL is trial-and-error based, meaning the agent discovers the best actions by trying out different options and refining its strategy based on the observed rewards.

Q-learning is a widely used model-free reinforcement learning algorithm that estimates the optimal action-value function, $Q(s,a)$ [watkins1992q]. The Q-function represents the expected cumulative reward the agent can obtain by taking action $a$ in state $s$ and then following the optimal policy. Q-learning uses a table to store Q-values and updates them iteratively based on the agent’s experiences. The learning process consists of the agent performing actions, receiving rewards, and updating the Q-values according to the following update rule:

|  | $Q(s,a)\leftarrow Q(s,a)+\alpha[r+\gamma\max_{a^{\prime}}Q(s^{\prime},a^{\prime% })-Q(s,a)]$ |  | (1) |

where $\alpha$ is the learning rate, $\gamma$ is the discount factor, $s^{\prime}$ is the new state after taking action $a$, and $r$ is the immediate reward received. The agent updates the Q-values to converge to their optimal values, which represent the expected long-term reward of taking each action in each state.

### II-D Motivating Example

Next, we illustrate the salient features of arat-rl using the Feature-Service specification as an example.

RL-based adaptive exploration. For the example in Figure [1](https://arxiv.org/html/2411.07098v1#S1.F1 "Figure 1 ‣ I Introduction ‣ Adaptive REST API Testing with Reinforcement Learning"), to perform the operation addFeatureToConfiguration, we must first create a product using a separate operation and establish a configuration for it using another operation. The sequence of operations should, therefore, be: create product, create configuration, and create feature name for the product with the specified configuration name. This example emphasizes the importance of determining the operation sequence. Our technique initially assigns weights to operations and parameters based on their usage frequency in the specification. In this case, productName is the most frequently used parameter across all operations; therefore, our technique assigns higher weights to operations involving productName. Specifically, the operation for creating a product gets the highest priority.

Moreover, once an operation is executed, its priority must be adjusted so that it is not explored repeatedly, creating new product instances unnecessarily. After processing a prioritized operation, our technique employs RL to adjust the weights in response to the API response received. If a successful response is obtained, negative rewards are assigned to the processed parameters, as our objective is to explore other uncovered operations. This method naturally leads to the selection of the next priority operation and parameter, facilitating efficient adjustments to the call sequence.

Inter-parameter dependencies [martin2019catalogue] can increase the complexity of the testing process, as some parameters might have mutual exclusivity or other constraints associated with them (e.g., only one of the parameters needs to be specified). RL-based exploration based in feedback received can also help with dealing with this complexity.

Dynamic construction of key-value pairs. One of the key steps in recent REST API testing techniques [Corradini2022, atlidakis2019restler, liu2022morest] is identification of producer-consumer relations between response schemas and request parameters. However, current tools face limitations when operations produce unstructured output (e.g., plain text) or have incomplete response schemas in their specifications. For instance, the addFeatureToConfiguration operation lacks structured response data (e.g., JSON format). Despite this, our approach processes and generates key-value data {productName: value, configurationName: value, featureName: value} from the request data, as the POST HTTP method indicates that a resource is created using the provided inputs.

By analyzing and storing key-value pairs identified from request and response data, even when the response schema is not explicitly provided or is incomplete, our dynamic key-value pair construction method proves especially beneficial in cases of responses with plain-text descriptions or incomplete response schemas. The technique can effectively uncover hidden dependencies not evident from the specification.

Sampling for efficient dynamic key-value pair construction. API response data can sometimes be quite large and processing every response for each key-value pair can be computationally expensive. To address this issue, we have incorporated a sampling-based strategy into our dynamic key-value pair construction method. This strategy efficiently processes the dynamic API feedback and adapts its exploration based on the gathered information while minimizing the overhead of processing every response.

![Refer to caption](img/57e1299f8179d9516ca8e07a5b719427.png)

Figure 2: Overview of our approach.

## III Our Approach

In this section, we introduce our Q-Learning-based REST API testing approach, which intelligently prioritizes and selects the operations, parameters, and value mapping sources while dynamically constructing key-value pairs. Figure [2](https://arxiv.org/html/2411.07098v1#S2.F2 "Figure 2 ‣ II-D Motivating Example ‣ II Background and Motivating Example ‣ Adaptive REST API Testing with Reinforcement Learning") provides a high-level overview of our approach. Initially, the Q-Learning Initialization module sets up the necessary variables and tables for the Q-learning process. The Q-Learning Updater subsequently receives these variables and tables and passes them to the Prioritization module, which is responsible for selecting operations, parameters, and value mapping sources.

Afterward, arat-rl sends a request to the System Under Test (SUT) and receives a response. It also supplies the request, response, selected operation, parameters, mapped value source, $\alpha$, $\gamma$, and $\epsilon$ to the Q-Learning Updater. The feedback is analyzed with the request and response, storing key-value pairs extracted from them for future use. The Updater component then adjusts the Q-values based on the outcomes, enabling the approach to adapt and refine its decision-making process over time. arat-rl iterates through this procedure until the specified time limit is reached. In the rest of this section, we present the details of the algorithm.

### III-A Q-Learning Table Initialization

The Q-Learning Table Initialization component, shown in Algorithm [1](https://arxiv.org/html/2411.07098v1#alg1 "Algorithm 1 ‣ III-A Q-Learning Table Initialization ‣ III Our Approach ‣ Adaptive REST API Testing with Reinforcement Learning"), is responsible for setting up the initial Q-table and Q-value data structures that guide the decision-making process throughout API testing. Crucially, this process happens without making any API calls.

The algorithm begins by setting the learning rate ($\alpha$) to 0.1, the discount factor ($\gamma$) to 0.99, and the exploration rate ($\epsilon$) to 0.1 (lines 2–4). These parameters control the learning and exploration process of the Q-Learning algorithm. The algorithm then initializes empty dictionaries for the Q-table and Q-value (lines 5–6). These parameters control the Q-Learning process, and the chosen values are those commonly recommended and used (e.g., [qlearningex1, qlearningex2, masadeh2018reinforcement]).

Algorithm 1 Q-Learning Table Initialization

1:procedure InitializeQLearning(operations)2:    Set learning rate ($\alpha$) to 0.13:    Set discount factor ($\gamma$) to 0.994:    Set discount factor ($\epsilon$) to 0.15:Initialize empty dictionary $q\_table$6:Initialize empty dictionary $q\_value$7:    for operation in operations do8:operation_id $\leftarrow$ operation[’operationId’]9:       $q\_value[\text{operation\_id}]\leftarrow$ new dictionary10:       for source in [1, 2, 3, 4, 5] do11:          $q\_value[\text{operation\_id}][\text{source}]\leftarrow$ 012:       end for13:       for parameter in operation[’parameters’] do14:param_name $\leftarrow$ parameter[’name’]15:          $q\_table[\text{param\_name}]=q\_table[\text{param\_name}]+1$16:       end for17:       for response_data in operation_data.get(’responses’) do18:          for key in response_data.keys() do19:             ifkey in $q\_table$ then20:                $q\_table[\text{key}]=q\_table[\text{key}]+1$21:             end if22:          end for23:       end for24:    end for25:    return  $\alpha$, $\gamma$, $\epsilon$, $q\_table$, $q\_value$26:end procedure

The algorithm iterates through each operation in the API (lines 7–24). For each operation, it extracts the operation’s unique identifier (operation_id) and initializes a new dictionary in the Q-value dictionary for the operation_id (lines 8–9). Next, it initializes the Q-values for each value mapping source (1 to 5) to 0 (lines 10–12).

The algorithm proceeds to iterate through each parameter in the operation (lines 13–16). It extracts the parameter’s name (param_name) and, if param_name already exists in the Q-table, it increments the corresponding entry by one. If it does not exist, it initializes this entry to one. This step builds the Q-table with a count of occurrences for each parameter.

Subsequently, the algorithm iterates through the response data of each operation (lines 17–23). It extracts keys from the response data and checks if the key is present in the Q-table for the operation_id (line 18). If the key is present in the Q-table for the operation_id, it increments the corresponding entry in the Q-table by 1 (lines 19–21). This step populates the Q-table with the frequency of occurrences for each response key as well.

Finally, the algorithm returns the learning rate ($\alpha$), discount factor ($\gamma$), exploration rate ($\epsilon$), Q-table, and Q-value dictionaries (line 25). This initial setup provides the Q-Learning algorithm with a foundational understanding of the API operations and their relationships, which is further refined during the testing process.

Algorithm 2 Q-Learning-based Prioritization

1:procedure SelectOperation2:Initialize $\text{max\_avg\_q\_value}\leftarrow-\infty$3:Initialize $\text{best\_operation}\leftarrow\text{None}$4:    for operation in operations do5:operation_id $\leftarrow$ operation[’operationId’]6:Initialize $\text{sum\_q\_value}\leftarrow 0$7:Initialize $\text{num\_params}\leftarrow\text{len(operation['parameters'])}$8:       for parameter in operation[’parameters’] do9:param_name $\leftarrow$ parameter[’name’]10:          $\text{sum\_q\_value}\leftarrow\text{sum\_q\_value}+q\_table[\text{param\_name}]$11:       end for12:       $\text{avg\_q\_value}\leftarrow\text{sum\_q\_value}/\text{num\_params}$13:       if $\text{avg\_q\_value}>\text{max\_avg\_q\_value}$ then14:          $\text{max\_avg\_q\_value}\leftarrow\text{avg\_q\_value}$15:          $\text{best\_operation}\leftarrow\text{operation}$16:       end if17:    end for18:    return  best_operation19:end procedure20:procedure SelectParameters(operation, $\epsilon$)21:Set $n$ randomly ($0\leq n\leq\text{length of operation['parameters']}$)22:Initialize empty list $selected\_parameters$23:    if $\text{random.random()}>\epsilon$ then24:       Sort operation[’parameters’] by Q-values in descending order25:       for $i\leftarrow 0$ to $n-1$ do26:Append operation[’parameters’][i] to $selected\_parameters$27:       end for28:    else29:       forparam in $\text{random.sample}(operation[^{\prime}parameters^{\prime}],n)$ do30:Append param to $selected\_parameters$31:       end for32:    end if33:    return  $selected\_parameters$34:end procedure35:procedure SelectValueMappingSource(operation, $\epsilon$)

Source1:

Example values in specification

Source2:

Random value generated by the parameter’s type and format

Source3:

Dynamically constructed key-value pairs from request

Source4:

Dynamically constructed key-value pairs from response

Source5:

Default values (string: string, number: 1.1, integer: 1, array: [], object: {})

36:operation_id $\leftarrow$ operation[’operationId’]37:    $sources\leftarrow[1,2,3,4,5]$38:    if $\text{random.random()}>\epsilon$ then39:       $max\_q\leftarrow\arg\max_{\text{source}\in sources}q\_value[\text{operation\_% id}][\text{source}]$40:       return  $max\_q$41:    else42:       return random.randint(1, 5)43:    end if44:end procedure

Algorithm 3 Q-Learning-based API Testing

1:procedure QLearningUpdater(response, $q\_table$, $q\_value$, selected_operation, selected_parameters, $\alpha$, $\gamma$)2:operation_id $\leftarrow$ selected_operation[’operation_id’]3:    if response.status_code is 2xx (successful) then4:       Extract key-value pairs from request and response5:reward $\leftarrow-1$6:Update $q\_value$ negatively7:    else if response.status_code is 4xx or 500 (unsuccessful) then8:reward $\leftarrow 1$9:Update $q\_value$ positively10:    end if11:    for $eachparaminselected\_parameters$ do12:       for $eachparam\_name,param\_valueinparam.items()$ do13:old_q_value $\leftarrow q\_table$[operation_id][param_name]14:max_q_value_next_state $\leftarrow$ max($q\_table$[operation_id].values())15:          $q\_table$[operation_id][param_name] $\leftarrow$ old_q_value + $\alpha$ * (reward + $\gamma$ * (max_q_value_next_state - old_q_value))16:       end for17:    end for18:    return  $q\_table$, $q\_value$19:end procedure20:procedure Main21:Initialize $\epsilon_{max}\leftarrow 1$22:Initialize $\epsilon_{adapt}\leftarrow 1.1$23:Initialize $time_{l}imit\leftarrow$ desired time limit in seconds24:operations $\leftarrow$ Load API specification25:    $\alpha$, $\gamma$, $\epsilon$, $q\_table$, $q\_value$  $\leftarrow$ InitializeQLearning(operations)26:    while Time Limit do27:best_operation $\leftarrow$ SelectOperation()28:selected_parameters $\leftarrow$ SelectParameters(best_operation, $\epsilon$)29:selected_source $\leftarrow$ SelectValueMappingSource(best_operation, $\epsilon$)30:response $\leftarrow$ Execute best_operation with selected_parameters and selected_source31:       $q\_table$, $q\_value$  $\leftarrow$ QLearningUpdater(response, $q\_table$, $q\_value$, selected_operation, selected_parameters, $\alpha$, $\gamma$)32:       $\epsilon\leftarrow\min(\epsilon_{max},\epsilon_{adapt}*\epsilon)$33:    end while34:end procedure

### III-B Q-Learning-based Prioritization

In this step, we prioritize API operations and select the best parameters and value mapping sources based on their Q-values. We present Algorithm [2](https://arxiv.org/html/2411.07098v1#alg2 "Algorithm 2 ‣ III-A Q-Learning Table Initialization ‣ III Our Approach ‣ Adaptive REST API Testing with Reinforcement Learning") (SelectOperation, SelectParameters, and SelectValueMappingSource) to describe the prioritization process.

The SelectOperation procedure (lines 1–19) is responsible for selecting the best API operation to test. The algorithm initializes variables to store the maximum average Q-value and the best operation (lines 2–3). It iterates through each operation, calculating the average Q-value for the operation’s parameters (lines 4–17). The operation with the highest average Q-value is selected as the best operation (line 15).

The SelectParameters procedure (lines 20–34) is responsible for selecting a subset of parameters for the chosen API operation. This selection is guided by the exploration rate ($\epsilon$). If a random value is greater than $\epsilon$, the algorithm selects the top $n$ parameters sorted by their Q-values in descending order (lines 23–27). Otherwise, the algorithm randomly selects $n$ parameters from the operation’s parameters (lines 28–31). The selected parameters are then returned (line 33).

The SelectValueMappingSource procedure (lines 35–44) is responsible for selecting the value mapping source for the chosen API operation. The technique leverages five types of value mapping sources.

*   •

    Source 1 (example values in specification): These values are provided in the API documentation as examples for a parameter. We consider three types of OpenAPI keywords that can specify example values: enum, example, and description [openapi]. Although the OpenAPI website mentions that users can specify example values in the description field, these examples are often not provided in a structured format but rather as natural language text. To extract example values from the description field, we create a list containing every word in the text, as well as every quoted phrase.

*   •

    Source 2 (random value generated by the parameter’s type, format, and constraints): This source generates random values for each parameter based on its type, format, and constraints. To generate random values, we utilize Python’s built-in random library. For date and date-time formats, we employ the datetime library to randomly select dates and times. If the parameter has a regular expression pattern specified in the API documentation, we generate the value randomly using the rstr library [rstr]. When a minimum or maximum constraint is present, we pass it to the random library to ensure that the generated values adhere to the specified constraints. This approach allows us to explore a broader range of values compared to the example values provided in the API specification.

*   •

    Source 3 (dynamically constructed key-value pairs from request): This source extracts key-value pairs from the dynamically constructed request key-value pairs. We employ Gestalt pattern matching [difflib] to identify the key most similar to the parameter name. This technique aids in discovering producer-consumer relationships.

*   •

    Source 4 (dynamically constructed key-value pairs from response): Similar to Source 3, this source obtains key-value pairs from dynamically constructed response key-value pairs. We use the same Gestalt pattern matching approach to identify the key, further assisting in the identification of producer-consumer relationships.

*   •

    Source 5 (default values): This source uses predefined default values for each data type (string: string, number: 1.1, integer: 1, array: [], object: {}). These default values can be useful for testing how the API behaves when it receives the simplest or most common input values.

Similar to the SelectParameters procedure, the selection of the value mapping source is guided by the exploration rate ($\epsilon$). If a random value is greater than $\epsilon$, the algorithm selects the mapping source with the highest Q-value for the chosen operation (lines 38–40). This helps the algorithm focus on the most promising mapping sources based on prior experience. Otherwise, the algorithm randomly selects a mapping source from the available sources (line 42). This randomness ensures that the algorithm occasionally explores less promising mapping sources to avoid getting stuck in a suboptimal solution.

### III-C Q-Learning-based API Testing

In this step, we execute the selected API operations with the selected parameters and value mapping sources, and update the Q-values based on the response status codes. Algorithm [3](https://arxiv.org/html/2411.07098v1#alg3 "Algorithm 3 ‣ III-A Q-Learning Table Initialization ‣ III Our Approach ‣ Adaptive REST API Testing with Reinforcement Learning") (QLearningUpdater and Main) describes the API testing process and the update of Q-values using the learning rate ($\alpha$) and discount factor ($\gamma$).

The QLearningUpdater procedure (lines 1–19) is responsible for updating the Q-values based on the response status codes. It first extracts the operation ID from the selected operation (line 2). If the response status code indicates a successful request (2xx), the algorithm extracts key-value pairs from the request and response, assigns a reward of $-1$, and updates the Q-values negatively (lines 3–6). If the response status code indicates an unsuccessful request (4xx or 500), the algorithm assigns a reward of 1 and updates the Q-values positively (lines 7–10). The Q-values are updated for each parameter in the selected parameters using the Bellman equation (lines 11–16), and the updated Q-values are returned (line 18).

The Main procedure (lines 20–34) orchestrates the Q-Learning-based API testing process. It initializes the exploration rate ($\epsilon$), its maximum value ($\epsilon_{max}$), its adaptation factor ($\epsilon_{adapt}$), and the desired time limit for testing (lines 21–23). The API specification is loaded and the Q-Learning table is initialized (lines 24–25). The algorithm then enters a loop that continues until the time limit is reached (line 26). In each iteration, the best operation, selected parameters, and selected mapping source are determined (lines 27–29). The API operation is executed with the selected parameters and mapping source, and the response is obtained (line 30). The Q-values are then updated based on the response (line 31), and the exploration rate ($\epsilon$) is updated (line 32).

By continuously updating the Q-values based on the response status codes and adapting the exploration rate, the Q-Learning-based API testing process aims to effectively explore the API operations and parameters, identifying potential issues in the API implementation.

## IV Evaluation

 Figure 3: Branch, line, and method coverage achieved by the tools on the benchmark services.

In this section, we present the results of empirical studies conducted to assess arat-rl. Our evaluation aims to address the following research questions:

1.  1.

    RQ1: How does arat-rl compare with state-of-the-art tools for REST API testing in terms of code coverage?

2.  2.

    RQ2: How does the efficiency of arat-rl, measured in terms of valid and fault-inducing requests generated and operations covered within a given time budget, compare to that of other REST API testing tools?

3.  3.

    RQ3: In terms of error detection, how does our approach perform at identifying 500 responses in REST APIs when compared to state-of-the-art REST API testing tools?

4.  4.

    RQ4: How do the main components of arat-rl—prioritization, dynamic key-value pair construction, and sampling—contribute to its overall performance?

### IV-A Experimental Setup

We performed our experiments using Google Cloud E2 machines, each equipped with 24-core CPU and 96 GB of RAM. We created a machine image containing all the services and tools in our benchmark. For each experiment, we deleted and recreated the machines using this image to minimize potential dependency issues. Each machine hosted all the services and tools under test, but we ran one tool at a time during the experiments. We monitored CPU and memory usage throughout the testing process to ensure that the testing tools were not affected by a lack of memory or CPU resources.

To evaluate the effectiveness and efficiency of our approach, we compared its against three state-of-the-art tools: EvoMaster [arcuri2019restful], RESTler [atlidakis2019restler], and Morest [liu2022morest]. We selected 10 RESTful services from a recent REST API testing study as our benchmark. We explain the selection process of these tools and services next.

#### Testing Tools Selection

As a preliminary note, because arat-rl is a black-box approach, we only considered black-box tools in our comparison. We believe that adding to the comparison white-box tools would result in an unfair comparison, as these tools leverage information about the code, rather than just information in the specification, to generate test inputs.

We identified an initial set of 10 tools based on a recent study [kim2022automated]. From this list, we chose (the black-box version of) EvoMaster [arcuri2019restful] and RESTler [atlidakis2019restler]. EvoMaster employs an evolutionary algorithm for automated test case generation and was the best-performing tool in that study and in another recent comparison [zhang2022open]. Its strong performance makes it an appropriate candidate for comparison. RESTler adopts a grammar-based fuzzing approach to explore APIs. It is a well-established tool in the field and, in fact, the most popular REST API testing tool in terms of GitHub stars.

Recently, two new tools have been published. Morest [liu2022morest] has been shown to have superior results compared to EvoMaster. We, therefore, included Morest in our set of tools for comparison, as it could potentially outperform the other tools. The other recent tool, RestCT, was also considered for inclusion in our evaluation. However, we encountered failures while running it. We contacted the authors, who confirmed the issues and said they will work on resolving them.

#### RESTful Services Selection

As benchmarks for our evaluation, we selected 10 out of 20 RESTful services from a recent study by Kim et al. [kim2022automated]. We had to exclude 10 services for various reasons. Specifically, we omitted the News service, developed by the author of our baseline EvoMaster, to avoid possible bias. Problem Controller and Spring Batch REST were excluded because they require specific domain knowledge to generate any meaningful test, so using them provides limited information about the tools. We excluded Erc20-rest-service and Spring Boot Actuator because some APIs in these services did not provide valid responses due to external dependencies being updated without corresponding updates in the service code. We also excluded Proxyprint, OCVN, and Scout API due to authentication issues that prevented them from generating meaningful responses. Finally, we excluded CatWatch and Cwa-verification because of their restrictive rate limits, which slowed down the testing process and made it impossible to collect results in a reasonable amount of time.

Our final set of services consisted of Features Service, LanguageTool, NCS, REST Countries, SCS, Genome Nexus, Person Controller, User Management Microservice, Simple Internet-Market, and Project Tracking System.

TABLE I: Comparison of operations covered and requests (2xx and 500 status codes) by arat-rl, Morest, EvoMaster, and RESTler.

 ARAT-RL Morest EvoMaster RESTler #operations #requests #operations #requests #operations #requests #operations #requests service covered 2xx&500 2xx 500 covered 2xx&500 2xx 500 covered 2xx&500 2xx 500 covered 2xx&500 2xx 500 Features Service 18 95,479 43,460 52,019 18 103,475 4,920 98,555 18 113,136 33,271 79,865 17 4,671 1,820 2,851 Language Tool 2 77,221 67,681 9,540 1 1,273 1,273 0 2 22,006 17,838 4,168 1 32,796 32,796 0 NCS 6 62,618 62,618 0 5 18,389 18,389 0 2 61,282 61,282 0 2 140 140 0 REST Countries 22 36,297 35,486 811 22 8,431 7,810 621 16 9,842 9,658 184 6 259 255 4 SCS 11 115,328 115,328 0 11 110,147 110,147 0 10 66,313 66,313 0 10 5,858 5,857 1 Genome Nexus 23 15,819 14,010 1,809 23 32,598 10,661 21,937 19 8,374 8,374 0 1 182 182 0 Person Controller 12 101,083 47,737 53,346 11 104,226 10,036 94,190 12 91,316 37,544 53,772 1 167 102 65 User Management 21 44,121 13,356 30,765 17 1,111 948 163 18 29,064 13,003 16,061 4 79 64 15 Market Service 12 29,393 6,295 23,098 6 1,399 394 1,005 5 10,697 4,302 6,395 2 1,278 0 1,278 Project Tracking 53 23,958 21,858 2,100 42 14,906 12,904 2,002 43 15,073 13,470 1,603 3 72 65 7 Average 18 60,132 42,783 17,349 15.6 39,595 17,748 21,847 14.5 42,710 26,505 16,205 4.7 4,550 4,128 422 

#### Result Collection

We ran each testing tool with a time budget of one hour per execution, as a previous study [kim2022automated] indicated that code coverage achieved by these tools tends to plateau within this duration. To accommodate for variability, we replicated the experiments ten times and calculated the average metrics across these runs.

Data collection for code coverage and status codes was performed using JaCoCo [jacoco] and Mitmproxy [mitmproxy], respectively. We focused on identifying unique instances of 500 errors, which generally signal server-side faults. The methodology was as follows:

1.  1.

    Stack Trace Collection: For services that provided stack traces accompanying 500 errors, we collected these traces, treating each unique trace as a separate fault. In the majority of cases, the errors we collected fall into this category.

2.  2.

    Response Text Analysis: In the absence of stack traces, we analyzed the response text. After removing unrelated components (e.g., timestamps), we classified unique instances of response text linked to 500 status codes as individual faults.

This systematic approach allowed us to compile a comprehensive and unique tally of faults for our analysis.

### IV-B RQ1: Effectiveness

To answer RQ1, we compared the tools in terms of branch, line, and method coverage achieved. Figure [3](https://arxiv.org/html/2411.07098v1#S4.F3 "Figure 3 ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning") presents the results of the study. The bar charts represent the coverage achieved by each tool for each RESTful service, whereas the boxplot at the bottom summarizes of each tool’s performance on the three coverage metrics across all the services.

As demonstrated in Figure [3](https://arxiv.org/html/2411.07098v1#S4.F3 "Figure 3 ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning"), arat-rl consistently outperforms the other tools in all three coverage metrics across the selected RESTful services. Our tool performed better, benefiting from operation, parameter, and value mapping prioritization, as seen in the motivating example. arat-rl is especially effective when there is operation dependency, parameter dependency, and value-mapping dependency. For example, the greatest coverage gains occurred for Language-tool, as shown in the figure, which has a complex set of inter-parameter dependencies and value-mapping dependencies. Meanwhile, arat-rl struggles with semantic parameters. For instance, it performed the worst for Market-service. The reason for this is that it requires input data, such as address, email, name, password, and phone number in specific formats, but arat-rl failed to generate valid values for these. Consequently, it was unable to create market users and use the user information for other operations in producer-consumer relationships.

On average, arat-rl attained 36.25% branch coverage, 58.47% line coverage, and 59.42% method coverage. In comparison, Morest, which exhibited the second-best performance, reached an average of 29.31% branch coverage, 52.27% line coverage, and 54.24% method coverage. Thus, the coverage gain of arat-rl over Morest is 23.69% for branch coverage, 11.87% for line coverage, and 9.55% for method coverage. EvoMaster and RESTler yield lower average coverage rates on all three metrics, with respective results of 26.45%, 48.37%, and 52.07% for EvoMaster and 16.54%, 36.58%, and 38.99% for RESTler for branch, line, and method coverage. The coverage gains of arat-rl compared to EvoMaster is 37.03% for branch coverage, 20.87% for line coverage, and 14.13% for method coverage, whereas compared to RESTler, it is 119.17% for branch coverage, 59.83% for line coverage, and 52.42% for method coverage.

These results provide evidence that our technique can effectively explore RESTful services, achieving superior code coverage compared to existing tools, and demonstrate its potential in addressing the challenges in REST API testing and enhancing the overall quality of software applications that rely on RESTful services.

<svg class="ltx_picture" height="73.07" id="S4.SS2.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,73.07) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">arat-rl consistently outperforms RESTler, EvoMaster, and Morest in terms of branch, line, and method coverage across the selected RESTful services. However, arat-rl can struggle with parameters that require inputs in specific formats.</foreignobject></g></g></svg>

TABLE II: Faults detected by the tools in 10 runs.

 |  | RESTler | EvoMaster | Morest | ARAT-RL |
| --- | --- | --- | --- | --- |
| Features-Service | 010 | 010 | 010 | 010 |
| Language-Tool | 000 | 048 | 000 | 122 |
| NCS | 000 | 000 | 000 | 000 |
| REST-Countries | 009 | 010 | 010 | 010 |
| SCS | 003 | 000 | 000 | 000 |
| Genome-Nexus | 000 | 000 | 005 | 010 |
| Person-Controller | 058 | 221 | 274 | 943 |
| User-Management | 010 | 010 | 008 | 010 |
| Market-Service | 010 | 010 | 010 | 010 |
| Project-Tracking | 010 | 010 | 010 | 010 |
| Total | 110 | 319 | 327 | 1125 | 

### IV-C RQ2: Efficiency

To address RQ2, we compared arat-rl to Morest, EvoMaster, and RESTler in terms of the number of (1) valid and fault-inducing requests generated (as indicated by HTTP status codes 2xx and 500) and (2) operations covered within a given time budget. Although efficiency is not only dependent on these metrics, due to factors such as API response time, we feel that they still represent meaningful proxies because they indicate the extent to which the tools are exploring the API and successfully identifying faults.

Table [I](https://arxiv.org/html/2411.07098v1#S4.T1 "TABLE I ‣ RESTful Services Selection ‣ IV-A Experimental Setup ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning") shows these metrics for ten different services: Features Service, Language Tool, NCS, REST Countries, SCS, Genome Nexus, Person Controller, User Management, Market Service, and Project Tracking. For each service, the table lists the number of operations covered and the number of requests made under the categories 2xx&500 (sum of 2xx code and 500 status code), 2xx, and 500, for each of the four tools. In the final row, the table presents the average number of operations covered and requests made by each tool across all the tested services.

In a given testing time budget of one hour, arat-rl generated more valid and fault-inducing requests, resulting in more exploration of the testing search space. Specifically, arat-rl generated 60,132 valid and fault-inducing requests on average, which is 52.01% more than Morest (39,595 requests), 40.79% more than EvoMaster (42,710 requests), and 1222% more than RESTler (4,550 requests).

This difference in the number of requests can be attributed to arat-rl’s approach of processing only a sample of key-value pairs from the response, rather than the entire response. By focusing on sampling key-value pairs, arat-rl efficiently identifies potential areas of improvement, contributing to a more effective REST API testing process.

Moreover, arat-rl covered more operations on average (18 operations) compared to Morest (15.6 operations), EvoMaster (14.5 operations), and RESTler (4.7 operations). This indicates that arat-rl efficiently generates more requests in a given time budget, which leads to covering more operations within the API, contributing to a more comprehensive testing process.

<svg class="ltx_picture" height="73.84" id="S4.SS3.p6.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,73.84) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="46.28" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">In a one-hour testing time budget, arat-rl outperforms Morest, EvoMaster, and RESTler by generating 52.01%, 40.79%, and 1222% more valid and fault-inducing requests respectively, and covering 15.38%, 24.14%, and 282.98% more operations.</foreignobject></g></g></svg>

### IV-D RQ3: Fault-Detection Capability

Table [II](https://arxiv.org/html/2411.07098v1#S4.T2 "TABLE II ‣ IV-B RQ1: Effectiveness ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning") shows the number of faults found by each tool in the selected RESTful services. As shown in Table [II](https://arxiv.org/html/2411.07098v1#S4.T2 "TABLE II ‣ IV-B RQ1: Effectiveness ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning"), arat-rl has the highest fault detection capability, with a total of 112.5 faults found across the selected RESTful services. In comparison, Morest and EvoMaster found 32.7 and 31.9 faults, respectively, while RESTler detected the lowest number of faults, with a total of 11.0\. arat-rl uncovered 9.2x, 2.5x, and 2.4x more bugs than RESTler, EvoMaster, and Morest, respectively.

Comparing this data against the data on 500 response codes from Table [I](https://arxiv.org/html/2411.07098v1#S4.T1 "TABLE I ‣ RESTful Services Selection ‣ IV-A Experimental Setup ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning"), we can see that, although arat-rl generated 20.59% fewer 500 responses, it found 250% more faults than Morest. Compared to EvoMaster, arat-rl generated 7.06% more 500 responses, but also detected 240% more faults. This indicates that arat-rl is more efficient at discovering faults via the exploration of diverse API states, whereas those tools tend to trigger the same failures more frequently.

arat-rl’s superior fault detection capability is particularly evident in the Language-Tool and Person-Controller services, where it detected 12.2 and 94.3 faults, respectively. These services have larger sets of parameters. For example, Language-Tool’s main operation /check, which checks text grammar, has 11 parameters. Similarly, Person-Controller’s main operation /api/person, which creates/modifies person instances, has 8 parameters. In contrast, the other services’ operations have three or fewer parameters.

arat-rl intelligently tries various parameter combinations with a reward system in Q-learning because we give negative rewards for the parameters in the successfully requested ones. This ability to explore various parameter combinations is a significant factor in revealing more bugs, especially in services with a larger number of parameters. These results indicate that arat-rl’s reinforcement-learning-based approach can effectively discover faults in RESTful services, providing valuable feedback to developers for improving software quality.

<svg class="ltx_picture" height="89.67" id="S4.SS4.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,89.67) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="62.11" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">arat-rl exhibits a superior fault-detection capability, uncovering 9.2x, 2.5x, and 2.4x more bugs than RESTler, EvoMaster, and Morest, respectively. This is mainly attributed to its intelligent RL-based exploration of various parameter combinations in services with a larger number of parameters.</foreignobject></g></g></svg>

TABLE III: Results of the ablation Study.

 Branch Line Method Faults Detected ARAT-RL 36.25 58.47 59.42 112.10 ARAT-RL (no prioritization) 28.70 (+26.3%) 53.27 (+9.8%) 55.51 0(+7%) 100.10 (+12%) ARAT-RL (no feedback) 32.69 (+10.9%) 54.80 (+6.9%) 56.09 (+5.9%) 110.80 (+1.2%) ARAT-RL (no sampling) 34.10 0(+6.3%) 56.39 (+3.7%) 57.20 (+3.9%) 112.50 (-0.4%) 

### IV-E RQ4: Ablation Study

To address RQ4, we conducted an ablation study to assess the impact of the main novel components of arat-rl on its performance. We compared the performance of arat-rl to three variants: (1) arat-rl without prioritization, (2) arat-rl without dynamic key-value construction from feedback, and (3) arat-rl without sampling. The results are presented in Table [III](https://arxiv.org/html/2411.07098v1#S4.T3 "TABLE III ‣ IV-D RQ3: Fault-Detection Capability ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning").

As illustrated in Table [III](https://arxiv.org/html/2411.07098v1#S4.T3 "TABLE III ‣ IV-D RQ3: Fault-Detection Capability ‣ IV Evaluation ‣ Adaptive REST API Testing with Reinforcement Learning"), the removal of any component results in reduction branch, line, and method coverage, as well as the number of found detected. Eliminating the prioritization component leads to the most substantial decline in performance, with branch, line, and method coverage decreasing to 28.70%, 53.27%, and 55.51%, respectively, and the number of found faults dropping to 100\. This evidence highlights the critical role of the prioritization mechanism in arat-rl’s effectiveness.

The absence of feedback and sampling components also negatively affects performance. Without feedback, arat-rl’s branch, line, and method coverage decreases to 32.69%, 54.80%, and 56.09%, respectively, and the number of found faults is reduced to 110.80\. Likewise, without sampling, branch, line, and method coverage drops to 34.10%, 56.39%, and 57.20%, respectively, while the number of found faults experiences a slight increase to 112.50, which may be attributed to random variations.

These findings emphasize that each component of arat-rl is essential for its overall effectiveness. The prioritization mechanism, feedback loop, and sampling strategy work together to optimize the tool’s performance in terms of code coverage and fault detection capabilities.

<svg class="ltx_picture" height="73.07" id="S4.SS5.p5.pic1" overflow="visible" version="1.1" width="600"><g fill="#000000" stroke="#000000" stroke-width="0.4pt" transform="translate(0,73.07) matrix(1 0 0 -1 0 0)"><g fill-opacity="1.0" transform="matrix(1.0 0.0 0.0 1.0 21.65 13.78)"><foreignobject color="#000000" height="45.51" overflow="visible" transform="matrix(1 0 0 -1 0 16.6)" width="556.69">Each component of arat-rl–prioritization, feedback, and sampling—contributes to arat-rl’s overall effectiveness. The prioritization mechanism, in particular, plays a significant role in enhancing arat-rl’s performance in code coverage achieved and faults detected.</foreignobject></g></g></svg>

### IV-F Threats to Validity

In this section, we address potential threats to our study’s validity and the steps taken to mitigate them. The internal validity is influenced by tool implementations and configurations, as well as the selected RESTful services. To minimize these threats, we used the latest tool versions, used default options, and chose a diverse set of 10 services for a fair comparison against state-of-the-art tools. Some testing tools might have randomness, but we addressed this issue by running each tool 10 times and computing the average results.

The external validity is affected by the limited number of RESTful services tested, impacting the generalizability of our findings. We tried to ensure a fair evaluation by selecting a diverse set of services, but future work should test a larger and more diverse set. Construct validity concerns the metrics and tool comparisons used. Metrics such as branch, line, and method coverage, number of requests, and 500 status codes as faults, although commonly, may not capture the test tools’ full quality. Additional metrics, such as mutation scores, could provide a better understanding. Including more tools or approaches in future studies would allow a more comprehensive evaluation of our technique’s performance.

In addition, we measured efficiency by considering the number of meaningful requests generated by the tools, including valid and fault-inducing ones, as well as the number of operations each tool covered within a given time budget. While these metrics give us some perspective on a tool’s performance, they do not consider all possible factors. Other aspects, such as the response times from the services, may also significantly impact overall efficiency.

## V Related Work

In this section, we provide an overview of related work in automated REST API testing, requirements based test case generation, and reinforcement learning based test case generation.

Automated REST API testing: EvoMaster [arcuri2019restful] is a white-box technique using evolutionary algorithms for test case generation, refining tests based on fitness and checking for 500 status codes. In contrast, black-box techniques include various tools with different strategies. RESTler [atlidakis2019restler] generates stateful test cases by inferring producer-consumer dependencies and targets 500 failures. RestTestGen [Corradini2022] exploits data dependencies and uses oracles for status codes and schema compliance. QuickREST [karlsson2020quickrest] is a property-based technique with stateful testing, checking non-500 status codes and schema compliance. RESTest [martin2020restest] accounts for inter-parameter dependencies, producing nominal and faulty tests with five types of oracles. Morest [liu2022morest] uses a dynamically updating RESTful-service Property Graph for meaningful test case generation. RestCT [wu2022combinatorial] employs Combinatorial Testing for RESTful API testing, generating test cases based on Swagger specifications.

Open-source black-box tools like Schemathesis [Zac2021schemathesis], Dredd [dredd], and Tcases [tcases] offer various testing capabilities. Schemathesis generates requests from input specifications, providing five types of oracles to check response compliance. Dredd tests REST APIs by comparing responses with expected results, validating status codes, headers, and body content. Tcases is a model-based tool that constructs an input space model from OpenAPI specifications, generating test cases covering valid input dimensions and checking response status codes for validation.

Requirements based test case generation: In requirements based testing, notable works include ucsCNL [barros2011ucscnl], which uses controlled natural language for use case specifications, and UML Collaboration Diagrams [badri2004use]. Requirements by Contracts [nebut2003requirements] proposed a custom language for functional requirements, while SCENT-Method [ryser1999scenario] employed a scenario-based approach with statecharts. SDL-based Test Generation [tahat2001requirement] transformed SDL requirements into EFSMs, and RTCM [yue2015rtcm] introduced a natural language-based framework with templates, rules, and keywords. These approaches typically deal with less formal requirements with the goal of generating test cases as accurately as possible according to the intended requirements. In contrast, arat-rl focuses on more formally specified REST APIs, which provide a structured basis for testing and effectively explore and adapt during the testing process to find faults.

Reinforcement learning based test case generation: Several recent studies have investigated the use of reinforcement learning in software testing, focusing primarily on web applications and mobile apps. These challenges often arise from hidden states, whereas in our approach, we have access to all states through the API specification but face more constraints on operations, parameters, and mapping values. Zheng et al.[zheng2021webrl] proposed an automatic web client testing approach utilizing curiosity-driven reinforcement learning. Pan et al.[pan2020androidrl] introduced a similar curiosity-driven approach for testing Android applications. Koroglu et al.[koroglu2018qbe] presented QBE, a Q-learning-based exploration method for Android apps. Mariani et al.[mariani2021autoblacktest] proposed AutoBlackTest, an automatic black-box testing approach for interactive applications. Adamo et al.[adamo2018androidrl] developed a reinforcement learning-based technique specifically for Android GUI testing. Vuong and Takada[vuong2018androidrl] also applied reinforcement learning to automated testing of Android apps. Köroğlu and Sen [koroglu2020androidrl] presented a method for generating functional tests from UI test scenarios using reinforcement learning for Android applications.

## VI Conclusion and Future Work

In this paper, we introduced arat-rl, a reinforcement learning-based approach for the automated testing of REST APIs. We assessed its effectiveness, efficiency, and fault-detection capability in comparison to state-of-the-art tools such as Morest, EvoMaster, and RESTler. Our experiments demonstrated arat-rl’s superior performance in terms of branch, line, and method coverage achieved, requests generated, and faults detected. We also conducted an ablation study, which highlighted the importance of arat-rl’s novel features—prioritization, dynamic key-value construction based on response data, and sampling from response data to speed up key-value construction—in enhancing arat-rl’s performance.

In future work, we will improve arat-rl’s performance by addressing its limitations in recognizing semantic constraints and generating inputs with specific formatting requirements, such as email, address, or phone number, as observed in our benchmark. We also plan to investigate ways to emulate complex dependency relationships such as those we observed for Morest. We will also study the impact of RL prioritization over time and investigate the effect of using different pattern-matching approaches. Additionally, we plan to extend our approach to support other types of web APIs, such as GraphQL or gRPC, by adapting our algorithms and testing framework to accommodate their specific requirements. Lastly, incorporating additional metrics and evaluation techniques, such as mutation testing, will enable a more comprehensive assessment of test case quality.

## Data availability

The artifact associated with this submission includes code, datasets, and other relevant material [artifact]. It can be used for reproducing the experiments presented in the paper and facilitating further research in this area.