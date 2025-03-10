---
layout: post
title:  "Using OpenSearch as the traces storage for Tracetest"
authors:
  - mathnogueira
date:   2022-09-14 12:15:00 -0300
categories:
  - community
---

It is exciting to announce that [Tracetest](https://tracetest.io) now supports OpenSearch as one of its trace backends. It means that if you already use OpenSearch Trace Analytics, you can start writing tests based on your telemetry using Tracetest without having to change anything on your application. In this article, we are going to explain how you can start trace-based tests right now.

### What is Tracetest

Tracetest is an open-source project that allows you to use your telemetry data to assert the behavior of you application. It fetches the telemetry generated by your application during a test execution and uses it to run assertions against it. This allows you to verify if the latencies of your system are within your thresholds, validate your application flow, and much more. It is also quite useful for writing end-to-end tests when you don't have a complete understanding of how the application interacts with other services. You can use the trace generated by a test to do exploration work while writing your tests.

### Installing Tracetest

The Tracetest team is putting a lot of effort trying to make installation as seamsless as possile. Right now, you can use their CLI to install the server to run either on top of Docker Compose or Kubernetes. Check their [installation guide](https://kubeshop.github.io/tracetest/installing/) to install both the CLI and server.

Before proceeding, make sure you have an OpenSearch instance running and have a data-prepper instance pointing to it. Data-prepper is an OpenSearch component capable of converting OpenTelemetry format into JSON that can be indexed by OpenSearch. Tracetest has an [example application](https://github.com/kubeshop/tracetest/tree/main/examples/tracetest-opensearch) where you can see how the Data Prepper is configured.

For simplicity sake, in this article, you will learn how to setup Tracetest using docker compose. Execute the following steps to configure your Tracetest instance to connect to your OpenSearch instance.

1. Run **tracetest version** to verify your CLI version
    1. Check if your CLI version is greater or equal to **0.7.2**
2. Create a folder called **tracetest-demo**
    ```sh
    mkdir ~/tracetest-demo
    ```
3. Open the folder on your terminal
    ```sh
    cd ~/tracetest-demo
    ```
4. Run **tracetest server install** and follow the steps:
    1. Select **Using Docker Compose**
    2. Allow the CLI to install any dependency that might be missing from your system
    3. Project's docker-compose file? **Press Enter**
    4. Do you want me to create an empty docker-compose file? **Yes**
    5. Do you have a supported tracing backend you want to use? (Jaeger, Tempo, OpenSearch, SignalFX) **Yes**
    6. Select **OpenSearch**
    7. Set the addresses to match your nodes addresses
      * Consider that the collector is running inside a docker container, so you cannot use `localhost` to reference other containers running on your machine. Use either `host.docker.internal` (Mac) or `172.17.0.1` (Linux) to reference your local machine.
    8. Set the index used to store your traces
    9. Set your data-prepper endpoint
      * Consider that the collector is running inside a docker container, so you cannot use `localhost` to reference other containers running on your machine. Use either `host.docker.internal` (Mac) or `172.17.0.1` (Linux) to reference your local machine.
    10. Do you have an OpenTelemetry Collector? **No**
    11. Do you want me to set up one? **Yes**
    12. Do you want to enable the demo app? **Yes**
    13. Tracetest output directory **Press Enter**
5. Run the **docker compose** command that the CLI just printed on your terminal
    ```sh
    docker compose -f docker-compose.yaml -f tracetest/docker-compose.yaml up -d
    ```

Now you should have Tracetest running and connected to your OpenSearch instance.

### Creating your first test

You should be able to access Tracetest by opening `http://localhost:8080` on your browser. You should see Tracetest's home screen

![Tracetest home page](/assets/media/tutorials/tracetest/home.png){: .img-fluid}

Click on **Create Test**, select **HTTP Request** and then click on **Next**

![Tracetest create http test](/assets/media/tutorials/tracetest/http_test.png){: .img-fluid}

Click on **Choose Example**, select **Pokemon - Add**, and then click on **Next**

![Tracetest create http test](/assets/media/tutorials/tracetest/create_test_from_example.png){: .img-fluid}

You can review the request that is used to trigger the test, you don't have to change anything. Just click on **Create**.

Now you can see the result of the test execution. You should be seeing the response got from the pokemon demo:

![Tracetest test trigger details](/assets/media/tutorials/tracetest/test_trigger.png){: .img-fluid}

Click on **Trace** and you should see the trace generated by the pokemon demo when the test was executed. This trace was retrieved from your OpenSearch instance.

![Tracetest test trace](/assets/media/tutorials/tracetest/test_trace.png){: .img-fluid}

And if you click on **Test** see all assertions for this test. However, as you just created the test, it doesn't have any assertion. You should add one to see how it works. Click on the **POST /pokemon** span and then click on **Add Test Spec**.

![Empty assertions](/assets/media/tutorials/tracetest/empty_assertions.png){: .img-fluid}

It will generate the query to select that specific span in the trace. And you can add assertions based on attributes from that span. Next test executions will run those assertions against the new trace automatically. To show its capabilities, add two assertions to that span.

Select **http.status_code** **Equals** **201** and it will assert the status code of the request to be 201 everytime this test is run.
Select **tracetest.span.duration** **Less than** **100ms** and it will assert that the request will take less than 100ms to run in future tests.

![Empty assertions](/assets/media/tutorials/tracetest/adding_assertions.png){: .img-fluid}

Now click on **Save Test Spec** and then on **Publish**.

Now it's possible to run the test and Tracetest will execute all assertions you just created using the new trace. Click on **Run Test** and wait for the test to finish executing.

![Adding assertions](/assets/media/tutorials/tracetest/test_rerun.png){: .img-fluid}


### Conclusion

With this new integration, you will be able to start using Tracetest to help you testing your application, and identify gaps in your telemetry that would only be discovered when you try to debug a problem. If you have any issues or questions about Tracetest, reach out the team on their [Discord channel](https://discord.gg/5mZm6bMx) or open an issue on their [git repository](https://github.com/kubeshop/tracetest).
