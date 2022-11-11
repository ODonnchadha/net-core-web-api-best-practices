## ASP.NET Core 6 Web API: Best Practices

- OVERVIEW:

- WHY CARE ABOUT BEST PRACTICES:
    - Versions: ASP.NET Core 6.0, .NET 6.0, Visual Studio 2022.
    - Best Practice: Via research and experience, produces optinmal results that are established or proposed as a suitable standard for widesread adaption.
    - Or, it all depends upon context.

- WEB API DESIGN BEST PRACTICES:
    - REST and resources:
        - Representations on resources using hypermedia. Representational state transfer.
            - Am architectural style for building distributed systems based on hypermedia and designed around resources.
            - Hypermedia, an extension of hypertext, is a nonlinear medium of information that includes graphics, audio, video, plain text, and hyperlinks.
            - Resources: any kind of object, data, or service that can be accessed by a client. Always has an unique indentifier which is an URI.
            - Resourse: Any information that can be named. REST uses an URI to indetify it and REST components transfer a representation of that resource.
        - URI naming. Use nouns. Use plurals. Append an identifier to specify an individual resource.
            - Follow the principle of least astonishment. And keep it simple and consistent.
        - Avoid exposing external details: Db table schema. Persistentence document format. Business domain entities. Leverage encapsulation.
    - HTTP Verbs:
        - GET: Request a resource. POST: Add a resource. PUT: Update a resource's state. 
        - DELETE: Delete a resource. PATCH: Update a subset of a resource's state.
        - HEAD: Request a resource but only return header data. (Improved performance.)
        - OPTIONS: Request information on how to work with a resource. (Supported server requests.)
        - With HTTP, idempotence and safety are the major attributes that seperate the HTTP methods.
            - Idempotence: Concept. 
            - Centain HTTP methods, aside from expiration or error, the side effects of one or more identical requests are the same as for a single request. 
            - We care more about the system. e.g.: elevator call buttons.
            - Queries. Nullipotent. No side effects.
        - Safe methods: GET and HEAD. Do not take action. Only retrieve. Can be prefetched. Agresively cached. Does not change system state.
        - GET       Idempotent? Yes.    Safe? Yes.
        - POST      Idempotent? No.     Safe? No.
        - PUT       Idempotent? Yes.    Safe? No.
        - DELETE    Idempotent? Yes.    Safe? No.
        - PATCH     Idempotent? Yes.    Safe? No.
    - HTTP Status Codes: Server responses to client requests.
        - Five classes: 1xx: Informational. 2xx: Successful (with some variation.) 3xx: Redirection. 4xx: Client error. 5xx: Server error.
        - Key scenarios: CRUD operations. (And security, permissions, and unhandled exceptions.)
            - 204: No content. (No avalable URI. Rare.)
            - 304: Not modified. Caching. Copy is still current.
            - 409: Conflict. (We are modifying a resource that has changed since client retrieval.)
            - 429: Too many requests.
            - Security & permissions:
                - 401: Unauthorized. (Really unathenticated.)
                - 403: Forbidden. (Authenticated but not having permissdion to access resource.)
                - 404: Not found.
    - Response Types: Built-in ASP.NET Core:
        - Return a specific type: List<Customer>
            - Primitives. Complex types. Collections. IEnumerables.
            - Pros: Strongly-typed. Simple.
            - Cons: Cannot easily return any other code or response type.
            - (And return an async<Task<T>> of that given type.)
            - NOTE: Use IAsyncEnumerable<T> to avoid buffering and guarentee asynchronous iteration where possible.
            - e.g.:
            ```csharp
                [HttpGet("products)]
                public async IAsyncEnumerable<Product> ListAsync()
                {
                    var products = this.repository.GetProductsAsync();
                    await foreach (var product in products)
                    {
                        if (product.IsOnSale)
                        {
                            yield return product;
                        }
                    }
                }
            ```
        - Return an IActionResult.
            - Pros: Works with built-in helper methods. Consistent return type for any method.
            - Cons: Not strongly-typed. Use atributes to inform Swagger.
        - Return ActionResult<T>:
            - RECOMMENDED.
            - Cons: Still need ProducesResponseType (unlike just returning a specific type.)
            - e.g.:
            ```csharp
                [HttpPost(), ProducesResponseType(StatusCodes.Status201Created)]
                public ActionResult<AuthorDto> Create(AuthorDto dto)
                {
                    var author = new Author { Name = dto.Name, TwitterAlias = dto.TwitterAlias };
                    // Add to database.
                    var createdAuthor = new AuthorDto(author.Id, author.Name, author.TwitterAlias);
                    return Created("authors/{id}, createdAuthor);
                }
            ```
        - Minimal APIs. IResult, Object, Raw JSON strings.
    - SUMMARY: 
        - REST. Resources. HTTP verbs. HTTP status codes. Returning ASP.NET Core response types.

- MODEL DESIGN BEST PRACTICES:
    - Web API Model Basics:
        - DTO: An object with no behavior. Only state.
        - POCO: An object that doesn't inherit from a type defined in an external dependency.
        - NOTE: DTOs are POCOs. (A POCO could have methods.)
        - API model types: Focused purely on transfering data. Can be C# records or classes. Can be annotated to support model validation.
            - Consider naming your DTOs in an intention-revealing manner. e.g.: CreateAuthorRequest versus Author.
    - The Robustness Principle (Postel's Law:)
        - Be conservative in what you send, and liberal in what you accept. (Via the TCP protocol.)
        - Hyrum's Law: With enough users, regardless of what you commit to in your contract, all observable behaviors of your system will be dependended upon by somebody.
        - "When you attempt to solve a problem with RegEx, now you have two problems."
        - BFF pattern. Backends for frontends. Structure APIs based upon client needs:
            - Single endpoint per data-driven screen.
            - Single endpoint per user-initiated command.
            - Don't expose APIs the client doesn't need to call.
            - Perform necessary data aggregation on the server.
            - Define summary and detailed representations of each resource.
            - Decide how to format collections of resources.
                - Filter support.
                - Paging support.
            - Decide how to format sub-collections of a resource.
        - What data should Web APIs return?
            - Provide exctly what the client(s) need. Optimal. Caching and local data storage. Ideal API call number is 1 to populate the entire screen.
        - Will your API support paging? Will you return page metadata and links in response bodies or response headers?
        - What index value, 0 or 1, will tour first page use? Paging variables?
    - Characteristics of good Web API models:
        - Well-named. (Models and URIs.) Avoid suffixes. Reveal intertion. Request or command. Response or result.
        - Well-factored. Avoid too many concepts. Seperate into distinct models. Avoid 1:1 relationships between resources when possible.
        - Well-sized. Tiny models are far too chatty. Large models are resource-intensive. Consider summary and detail representations. 
            - Provide paging and filtering.
        - Stable models. Rare model changes. Ideally, only add data. Use intension-specific models.
    - Web API Model antipatterns:
        - Avoid non-standard names. Avoid database concept leakage. Avoid breaking changes without versioning. Offer client control over reponse size. 
            - Provide documentation. e.g. Swagger via Swashbuckle.
    - Summary:
        - Web API basics. DTOs. with only needed data. Robustness. Sumary versus detail. Paging consideration. Avoid anti-patterns.

- WEB API IMPLEMENTATION BEST PRACTICES IN ASP.NET CORE:
    - ASP.NET Core MVC:
        - Decorate controllers with the following.
        ```csharp
            [ApiController()]
        ```
            - Attrribute routing. Model binding and validation. Minimize controller size. Use filters to apply policies. async/await.
            - Response caching support. Response compression. Content negotiation. Default exception handler. (development versus production.)
            - ActionResult<T>. ControllerBase base class.
        - Keep controllers small and easy to manage.
            - Use filters for duplicate logic. Avoid business logic. Avoid data access logic. Avoid try/catch.
            - Large controllers lack cohesion. Grow ever larger. Have more total dependencies and coupling.
        - Use filters and middleware:
            - Repetitive code in actions/endpoints can be refactored into filters.
            - If it doesn't need MVC details, use middleware instead.
            - Filters and middleware equals consistent policy. Model validation. Logging. Analytics.
        - async/await should be the default, returning a Task<T>.
        - Use [ResponseCache] headers. (Not stored on the server by default.)
            - Add response caching middleware. (With VaryByParam options.) Stores in server memory. (Added to any controller action.)
        - Response compression. Rely on server compression by default. Do not compress small responses due to overhead.
            - Clients inform the server with "Accept-Encoding" header. Send "Content-Encoding" header with response.
            - Use caution with compression over HTTPS.
            ```csharp
                var builder = WebApplication.CreateBuilder(args);
                builder.Services.AddResponseCompression(options =>
                {
                    options.EnableForHttps = true;
                });
                var app = builder.Build();
                app.UseResponseCompression();
            ```
            - Production error handling:
            ```csharp
                [ApiExplorerSettings(IgnoreApi = true), Route("/error")]
                public IActiuonResult HandleError() => Problem();
            ```
            - Development error handling:
            ```csharp
                [ApiExplorerSettings(IgnoreApi = true), Route("/error-development")]
                public IActiuonResult HandleError([FromServices]IHostEnvironment env)
                {
                    if (!env.IsDevelopment() return NotFound();
                    var ex = HttpContext.Features.Get<IExceptionHandlerFeature>()!;
                    return Problem(ex.Error.StackTrace, ex.Error.Message);
                }
            ```
    - Using MediatR with Web APIs:
        - Open-source Mediator design pattern.
        - Mediator promotes loose-coupling by keeping objects from refering to each other explicitly, and allows variance with thier independent interaction.
            - Interfaces: IRequest<TResponse>. IRequestHandler<TRequest,TResponse>.
            - Scans assemblies upon start. And IMediator type and imlementation.
                1. Calling _mediator.Send(request).
                2. Finds the handler for typeof(request).
                3. Invokes the handler.
                4. Returns the TResponse result.
            - No compile-time dependency/knowledge of the code handling the request. Notifications as well with multiple handlers.
        - Typical API controller:
            1. Create(DTO) singnature.
            2. Validate the DTO.
            3. Map DTO to entity.
            4. Use abstraction to persist entity.
            5. Map entity to DTO.
            6. return appropriate HTTP result.
        ```csharp
            [HttpPost(), ProducesResponseType(StatusCodes.Status201Created, Type=(typeof(AuthorDto))]
            public async Task<ActionResult<AuthorDto>> Create(AuthorDto dto, CancellationToken token)
            {
                var entity = new Author { name = dto.Name, TwitterAlias = dto.TwitterAlias };
                await repository.AddAsync(author, token);
                var model = new AuthorDto(author.Id, author.Name, author.TwitterAlias);

                return Created($"authors/{model.Id}", model);
            }
        ```
        - Mediator:
        ```csharp
            public IMediator Mediator { get; }
            ctor(IMediator mediator) => Mediator = mediator;
            [HttpPost(), ProducesResponseType(StatusCodes.Status201Created, Type=(typeof(AuthorDto))]
            public async Task<ActionResult<AuthorDto>> Create(AuthorDto dto, CancellationToken token)
            {
                var command = new CreateAuthorCommand(dto.Name, dto.TwitterAlias);
                var model = await Mediator.Send(command, token);
                return Created($"authors/{model.Id}", model);
            }
        ```
    - Moving from Controllers to Endpoints:
        - Requests routed directly to the "handler." While still building on top of ASP.NET MVC Core.
        - [Sales Pitch.]
    - Minimal API best practices:
        - Alternative to controllers and the MVC pattern. Lighweight. Multi-file approach for a non-trivial application.
        - IActionResult versus IResult. Minimal APIs work with Swagger.
        ```csharp
            [HttpPost("/payments")]
            public IActionResult Post([FromBody]PaymentRequest? request)
            {

            }
        ```
        ```csharp
            app.MapPost("/payments", (PaymentRequest request) => { });
        ```
        - Big challenge. Organization. A large program.cs file is a bad idea. Use folders to organize endpoints by resource.
        - Use infdividual files per endpoint.
    - Adding Background Services to Web APIs:
        - Stop-gap to avoid multiple hosted services.
        - Configure a HostedService in your app with AddHistedService<T>.
        - Define the service as a type inheriting from BackgroundService.
        - Override the ExecuteAsync method to perform the work.
        - Specify a delay to control execution frequency.
        ```csharp
            builder.Services.AddHostedService<DataConsistencyWorker>();
            builder.Services.AddScoped<IScopedProcessingService, ScopedProcessingService>();
            public class DataConsistencyWorker : BackgroundService
            {
                public IServiceProvider Service { get; }
                // Use this to manage scope within background service (singleton.)
                public DataConsistencyWorker(IServiceProvider service) => Service = service;
                protected override async Task ExecuteAsync(CancellationToken token)
                {
                    await Execute(token);
                }
                private async Task ExecuteAsync(CancellationToken token)
                {
                    // Create the scope itself.
                    using (var scope = Service.CreateScope())
                    {
                        var service = scope.ServiceProvider.GetRequiredService<IScopedProcessingService>();
                        await service.ExecuteAsync(token)
                    }
                }
                public interface IScopedProcessingService
                {
                    Task ExecuteAsync(CancellationToken token);
                }
                public class ScopedProcessingService : IScopedProcessingService
                {
                    public async Task ExecuteAsync(CancellationToken token)
                    {
                        // ExecuteAsync
                        await Task.Delay(30000, token);
                    }
                }
            } 
        ````
    - Summary:
        - Best practices: ASP.NET Core MVC. MediatR. Minimal APIs. Background Serives with Web APIs.

- SECURITY BEST PRACTICES:
    - JWTs:
    - Using JWTs properly:
    - Alternatives to JETs:
    - Implementing authorization/authentication in ASP.NET Core:
    - COnfiguroing CORS & HTTPS:

- TESTING BEST PRACTICES: