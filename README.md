# Create a Rust function in Azure using VS Code

You use Visual Studio Code to create a custom handler function 
that responds to HTTP requests. After testing the code locally, 
you deploy it to the serverless environment of Azure Functions.
Custom handlers can be used to create functions in any language 
or runtime by running an HTTP server process. This example supports Rust.

Completing this quickstart incurs a small cost of a few USD cents or less in your Azure account.
<img width="1680" alt="Screenshot 2565-11-11 at 21 54 24" src="https://user-images.githubusercontent.com/15844801/201365763-3cf95729-3ea2-4022-93df-c099b2c30e39.png">

<img width="936" alt="Screenshot 2565-11-11 at 21 57 39" src="https://user-images.githubusercontent.com/15844801/201366308-7b4bff6a-70a6-41ad-af43-ca10bb4e02ce.png">

## Create your local project

In this section, you use Visual Studio Code to create a local Azure Functions custom handlers project. 
Later in this article, you'll publish your function code to Azure.

### Configure your Rust environment

Before you get started, make sure you have the following requirements in place:

+ An Azure account with an active subscription. [Create an account for free](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Visual Studio Code](https://code.visualstudio.com/) on one of the [supported platforms](https://code.visualstudio.com/docs/supporting/requirements#_platforms).

+ The [Azure Functions extension](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurefunctions) for Visual Studio Code.

+ The [Azure Functions Core Tools](./functions-run-local.md#v2) version 3.x. Use the `func --version` command to check that it is correctly installed.

+ Rust toolchain using [rustup](https://www.rust-lang.org/tools/install). Use the `rustc --version` command to check your version.

---

1. Choose the Azure icon in the Activity bar. Then in the Workspace (local) area, select the + button, 
choose Create Function in the dropdown. When prompted, choose Create new project.
![image](https://user-images.githubusercontent.com/15844801/201353828-a2badb03-2fc2-4c25-8f4c-b5f9f9171d2e.png)
1. Choose the directory location for your project workspace and choose Select. You should either 
create a new folder or choose an empty folder for the project workspace. Don't choose a project folder 
that is already part of a workspace.
1. Provide the following information at the prompts:
    |Prompt|Selection|
    |--|--|
    |**Select a language for your function project**|Choose `Custom Handler`.|
    |**Select a template for your project's first function**|Choose `HTTP trigger`.|
    |**Provide a function name**|Type `HttpHandler`.|
    |**Authorization level**|Choose `Anonymous`, which enables anyone to call your function endpoint. To learn about authorization level, see [Authorization keys](functions-bindings-http-webhook-trigger.md#authorization-keys).|
    |**Select how you would like to open your project**|Choose `Add to workspace`.|

## Create and build your function

The *function.json* file in the *HttpExample* folder declares an HTTP trigger function. You complete the function by adding a handler and compiling it into an executable.

### Rust Code

1. Press <kbd>Ctrl + Shift + `</kbd> or select *New Terminal* from the *Terminal* menu to open a new integrated terminal in VS Code.

1. In the function app root (the same folder as *host.json*), initialize a Rust project named `handler`.

    ```bash
    cargo init --name handler
    ```

1. In *Cargo.toml*, add the following dependencies necessary to complete this quickstart. The example uses the [warp](https://docs.rs/warp/) web server framework.

    ```toml
    [dependencies]
    warp = "0.3"
    tokio = { version = "1", features = ["rt", "macros", "rt-multi-thread"] }
    ```

1. In *src/main.rs*, add the following code and save the file. This is your Rust custom handler.

    ```rust
    use std::collections::HashMap;
    use std::env;
    use std::net::Ipv4Addr;
    use warp::{http::Response, Filter};

    #[tokio::main]
    async fn main() {
        let handler1 = warp::get()
            .and(warp::path("api"))
            .and(warp::path("HttpTriggerHandler"))
            .and(warp::query::<HashMap<String, String>>())
            .map(|p: HashMap<String, String>| match p.get("name") {
                Some(name) => Response::builder().body(format!("Hello, {}. This HTTP triggered function executed successfully.", name)),
                None => Response::builder().body(String::from("This HTTP triggered function executed successfully. Pass a name in the query string for a personalized response.")),
            });

        let port_key = "FUNCTIONS_CUSTOMHANDLER_PORT";
        let port: u16 = match env::var(port_key) {
            Ok(val) => val.parse().expect("Custom Handler port is not a number!"),
            Err(_) => 3000,
        };

        warp::serve(handler1).run((Ipv4Addr::LOCALHOST, port)).await
    }
    ```

1. Compile a binary for your custom handler. An executable file named `handler` (`handler.exe` on Windows) is output in the function app root folder.

    ```bash
    cargo build --release
    cp target/release/handler .
    ```

   ![image](https://user-images.githubusercontent.com/15844801/201355938-d0e3fde1-11ef-411c-937a-bcf0788e4f8f.png)


---

## Configure your function app

The function host needs to be configured to run your custom handler binary when it starts.

1. Open *host.json*.

1. In the `customHandler.description` section, set the value of `defaultExecutablePath` to `handler` (on Windows, set it to `handler.exe`).

1. In the `customHandler` section, add a property named `enableForwardingHttpRequest` and set its value to `true`. For functions consisting of only an HTTP trigger, this setting simplifies programming by allow you to work with a typical HTTP request instead of the custom handler [request payload](functions-custom-handlers.md#request-payload).

1. Confirm the `customHandler` section looks like this example. Save the file.

    ```
    "customHandler": {
      "description": {
        "defaultExecutablePath": "handler",
        "workingDirectory": "",
        "arguments": []
      },
      "enableForwardingHttpRequest": true
    }
    ```

The function app is configured to start your custom handler executable.

## Run the function locally

You can run this project on your local development computer before you publish to Azure.

1. In the integrated terminal, start the function app using Azure Functions Core Tools.

    ```bash
    func start
    ```

2. With Core Tools running, navigate to the following URL to execute a GET request, which includes `?name=Functions` query string.

    `http://localhost:7071/api/HttpExample?name=Functions`

3. A response is returned, which looks like the following in a browser:
![image](https://user-images.githubusercontent.com/15844801/201356858-c93eff89-9b67-4ee0-8201-a26933dff59f.png)

4. Information about the request is shown in **Terminal** panel.
<img width="1296" alt="Screenshot 2565-11-11 at 21 08 22" src="https://user-images.githubusercontent.com/15844801/201357057-1665783b-98c2-4f0c-a4e7-f9d9d955c4e2.png">

5. Press <kbd>Ctrl + C</kbd> to stop Core Tools.

After you've verified that the function runs correctly on your local computer, it's time to use Visual Studio Code to publish the project directly to Azure.

## Sign in to Azure

Before you can publish your app, you must sign in to Azure.

1. If you aren't already signed in, choose the Azure icon in the Activity bar. Then in the Resources area, choose Sign in to Azure....
![image](https://user-images.githubusercontent.com/15844801/201359427-ed09c66c-20d9-402d-96d8-fe41e2d7cb11.png)
If you're already signed in and can see your existing subscriptions, go to the next section. If you don't yet have an Azure account, choose Create and Azure Account.... Students can choose Create and Azure for Students Account....
1. When prompted in the browser, choose your Azure account and sign in using your Azure account credentials. If you create a new account, you can sign in after your account is created.

1. After you've successfully signed in, you can close the new browser window. The subscriptions that belong to your Azure account are displayed in the sidebar.


## Compile the custom handler for Azure

In this section, you publish your project to Azure in a function app running Linux. In most cases, you must recompile your binary and adjust your configuration to match the target platform before publishing it to Azure.

1. Create a file at *.cargo/config*. Add the following contents and save the file.

    ```
    [target.x86_64-unknown-linux-musl]
    linker = "rust-lld"
    ```

1. In the integrated terminal, compile the handler to Linux/x64. A binary named `handler` is created. Copy it to the function app root.

    ```bash
    rustup target add x86_64-unknown-linux-musl
    cargo build --release --target=x86_64-unknown-linux-musl
    cp target/x86_64-unknown-linux-musl/release/handler .
    ```

1. If you are using Windows, change the `defaultExecutablePath` in *host.json* from `handler.exe` to `handler`. This instructs the function app to run the Linux binary.

1. Add the following line to the *.funcignore* file:

    ```
    target
    ```

    This prevents publishing the contents of the *target* folder.

---

## Create the function app in Azure

In this section, you create a function app and related resources in your Azure subscription. 

1. Choose the Azure icon in the Activity bar. Then in the **Resources** area, select the **+** icon and choose the **Create Function App in Azure** option.

    ![image](https://user-images.githubusercontent.com/15844801/201359795-cdda4a7c-3cd2-48d3-a279-0c77dca8e671.png)


2. Provide the following information at the prompts:

    |Prompt|Selection|
    |--|--|
    |**Select subscription**| Choose the subscription to use. You won't see this when you have only one subscription visible under **Resources**. |
    |**Enter a globally unique name for the function app**| Type a name that is valid in a URL path. The name you type is validated to make sure that it's unique in Azure Functions.|
    |**Select a runtime stack**| Choose **Custom Handler**. |
    |**Select a location for new resources**| For better performance, choose a [region](https://azure.microsoft.com/regions/) near you.|

    The extension shows the status of individual resources as they are being created in Azure in the **Azure: Activity Log** panel.

![image](https://user-images.githubusercontent.com/15844801/201359828-0618e27d-cb65-4708-960d-f88739cd4a10.png)


3. When the creation is complete, the following Azure resources are created in your subscription. The resources are named based on your function app name:

    - A resource group, which is a logical container for related resources.
    - A standard Azure Storage account, which maintains state and other information about your projects.
    - A function app, which provides the environment for executing your function code. A function app lets you group functions as a logical unit for easier management, deployment, and sharing of resources within the same hosting plan.
    - An App Service plan, which defines the underlying host for your function app.
    - An Application Insights instance connected to the function app, which tracks usage of your functions in the app.


## Deploy the project to Azure

1. Choose the Azure icon in the Activity bar, then in the Workspace area, select your project folder and select the Deploy... button.
![image](https://user-images.githubusercontent.com/15844801/201359990-c5e4ebf9-c462-4944-b11f-bdb805cfcf1c.png)
1. Select Deploy to Function App..., choose the function app you just created, and select Deploy.
1. After deployment completes, select View Output to view the creation and deployment results, including the Azure resources that you created. If you miss the notification, select the bell icon in the lower right corner to see it again.

![image](https://user-images.githubusercontent.com/15844801/201360056-447d5ebf-4c94-4fcc-b6a5-643dfd587fa9.png)

## Run the function in Azure

1. Back in the Resources area in the side bar, expand your subscription, your new function app, and Functions. Right-click (Windows) or Ctrl - click (macOS) the HttpExample function and choose Execute Function Now....
![image](https://user-images.githubusercontent.com/15844801/201360184-b82d027f-6972-46e0-a065-65ffb554017f.png)
1. In Enter request body you see the request message body value of { "name": "Azure" }. Press Enter to send this request message to your function.

1. When the function executes in Azure and returns a response, a notification is raised in Visual Studio Code.

## Clean up resources

When you continue to the next step and add an Azure Storage queue binding to your function, you'll need to keep all your resources in place to build on what you've already done.

Otherwise, you can use the following steps to delete the function app and its related resources to avoid incurring any further costs.

1. In Visual Studio Code, press F1 to open the command palette. In the command palette, search for and select Azure: Open in portal.

1. Choose your function app and press Enter. The function app page opens in the Azure portal.

1. In the Overview tab, select the named link next to Resource group.
![image](https://user-images.githubusercontent.com/15844801/201360441-7df97a60-7086-48f8-b426-c5f4e98ff4bc.png)

1. On the Resource group page, review the list of included resources, and verify that they're the ones you want to delete.

1. Select Delete resource group, and follow the instructions.

Deletion may take a couple of minutes. When it's done, a notification appears for a few seconds. You can also select the bell icon at the top of the page to view the notification.
