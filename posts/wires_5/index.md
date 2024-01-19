# Wires #5: Exploring the .NET Web API and enable HTTPs [Backend]

Let's explore dotnet everyone :D

<!--more-->

## How to use Environment Variables
How to use the Environment Variables to allow separate production from development

## How enable SSL on the application
As you know I issued a SSL Certificate using the certbot which uses `let's encrypt`, it took me so much time to figure this out as the docs aren't clear at all.

{{< image src="ssl_errors.png" position="center" style="border-radius: 8px;" caption="I wasted 2 days trying to fix that, and I even gave up and switched to Go!" >}}

Certbot generates some `.pem` files, which is used by almost all the apis expect .NET or Idk how to use them with `Kestrel`. Anyway certs and keys need to be converted from `.pem` file to `.pfx`.

```
sudo openssl pkcs12 -export -out cert.pfx \
    -inkey /etc/letsencrypt/live/<domain>/privkey.pem \
    -in /etc/letsencrypt/live/<domain>/fullchain.pem
```

It will ask for a password which is used later to configure `Kestrel`.

```csharp
// enable https
builder.WebHost.UseKestrel(options => {
    options.ListenAnyIP(443, listenOptions => {
        listenOptions.UseHttps("cert.pfx", "password");
    });
});
```

PS: I hate how C# loves to waste a new line for `{}`

## Working with the dotnet CLI
Navigate to the [docs](https://learn.microsoft.com/en-us/dotnet/core/tools/) to learn more about the dotnet CLI.

- How to create a template webapi with controllers : `dotnet new webapi --use-controllers -n backend`
- Build vs Publish 

`Building` is the process of compiling the source code of your application into executable files or libraries. When you build a .NET application, the compiler translates your source code (written in C#, F#, or another supported language) into Intermediate Language (IL) code, which is then packaged into assemblies.

The build process involves several steps:

1. Compilation: The compiler translates the source code into IL code.
2. Assembly Generation: The IL code is packaged into assemblies, which may include executable files (e.g., .exe for console applications) and/or dynamic link libraries (DLLs).
3. Dependency Resolution: Dependencies are resolved, and required libraries are referenced.
4. Output Generation: The compiled application or library is generated as output.

`Publishing` is the process of preparing and packaging your application for deployment. It involves not only building the application but also including all necessary files and dependencies to ensure that the application can run on the target environment.

The publish process includes the following steps:

1. Build: The application is built, following the same steps as in the build process.
2. Dependency Packaging: All required dependencies (including .NET runtime components) are packaged with the application.
3. Configuration Transformation: Configuration files may be transformed based on the target environment.
4. Output Packaging: The application is packaged into a directory or a single executable file.

## Testing the API
Before this step, I created a domain called `api` which I will link the application to server on it using HTTPs (Make sure to also create a cert if you haven't).

Let's publish and run the server!
1. Publish: `dotnet publish`
2. Run: Make run that the server is running with `sudo` as for ports lower than `1024` : `sudo dotnet backend.dll`

{{< image src="https.png" position="center" style="border-radius: 8px;" caption="Now API works with HTTPs" >}}

