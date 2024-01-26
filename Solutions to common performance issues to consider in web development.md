# Ideas to solve some performance issues in web development

Some ideas may be appropriate or not for you depending of a lot of other factors, such as: negatively impact maintenability of the code, having more code to write, more complex code, lower time to market, not working on adding features, etc. You need to find the right balance. But, more importly, do not blindly apply most of the ideas. Evaluate your application, find the most important bottlenecks and apply the ideas having the most positive impact for you. In all case, monitor and track the activity. Software tends to accumulate data and performance issues arise progressively with time in various ways.

## General tips
- Use monitoring tool (e.g. Application Insights) to find slow endpoint, slow SQL request, endpoints doing to much SQL queries or similar one, etc.
- Consider adding cache layer(s) to avoid redoing treatment and respond quickly (redis, in memory structure, can be done when starting the app, etc.)
- Reusing object to minimize impact on language with garbage collector

## Front-end
- Back-end pagination instead of being done in the front-end
- Calls to the API one after the other instead of making them in parallel when there is no dependency

## Back-end
- Delaying operations. The endpoint can acknowledge having received the request and processing it later.
- Some informations can be precalculated once or based on a schedule
- Using known algorithms to solve specific problem:
    such as:
    - BloomFilter. A space-efficient probabilistic data structure that is used to test whether an element is a member of a set. It will return either "probably in set" or "definitely not in a set". It can be used to limit the overall number of operation. (SQL call, api call, file access, etc.)
    - LRU. To store the most used entries in a cache and reduce the overall number of operation and/or delay to fetch the data.
    - Aho–Corasick. A string-searching algorithm that locates elements of a finite set of strings (the "dictionary") within an input text. First, it will build a tree structure of the set of strings. Then, when searching, it will navigate within the tree that represent part of a string. So, the complexity of the algorithm is linear in the length of the strings plus the length of the searched text plus the number of output matches. In other word, the set of strings can be really huge and it will not impact the run time.
- The choice of data structure (list vs dictionary vs hash set) and the type of operation performed. The time to prepare the structure can help or hurt the run time.
- Reducing amount of memory used at any time. The data can be read in streaming from the database or the services.
- Reduce excessive object mapping. ex: Map repository entities to "business" objects, then map them to other objects "DTO" for the response of the endpoint

## C#
- HttpClient
    Even though the HttpClient class is disposable, it should not be disposed directly because the associated socket is not immediately freed. Having many HttpClient instances can cause you to run out of sockets and slow down a web application. The HttpClient class is thread safe and it might be tempting to assign it to a static instance field for reuse. At that point, we fix the problem, but we create another one. It no longer takes into account changes at the DNS level. One solution is to use IHttpClientFactory. (ref: https://learn.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)
- Minimize object creation (e.g.: using StringBuilder for concatenation of string)
- Minimize use of regex (on a side note: by relaying on too much data validation, you may fall into a falsehood programmer believe in. ref: https://github.com/kdeldycke/awesome-falsehood)
- Consider putting a global timeout on regex to prevent excessive CPU usage on complex regex or being stuck processing the regex (an issue named: Catastrophic Backtracking)
- Using library that are using source generators in order to avoids the need for reflection at runtime
- Consider using Native AOT

### Entity Framework
- Investigate lazy loading with EF (aka "n + 1 problem"). It can causes lots of SQL queries. A large number of queries causes a slowdown and conversely a query that is too complex can be longer. Consider: https://learn.microsoft.com/en-us/ef/core/querying/related-data/eager?source=recommendations#filtered-include or to group certain calls. For example, instead of processing one entity at a time. This could be done in batches. So fewer SQL queries. A suggestion for a starting point, use the monitoring tool to find endpoints having excessive number of SQL Query. It can be a good starting point. Then, use an interceptor to find the exact place in the code.

    Here is an example with EF core:

    - Add in the Startup class: `DiagnosticListener.AllListeners.Subscribe(new EfGlobalListener());`
    - Then add the following file:
    
    ```
    public class EfGlobalListener : IObserver<DiagnosticListener>
    {
        private readonly EfInterceptor _interceptor = new();
 
        public void OnCompleted() { }
 
        public void OnError(Exception error) { }
 
        public void OnNext(DiagnosticListener listener)
        {
            if (listener.Name == DbLoggerCategory.Name)
                listener.Subscribe(_interceptor);
        }
    }
    
    public class EfInterceptor : IObserver<KeyValuePair<string, object>>
    {
        public void OnCompleted() { }

        public void OnError(Exception error) { }

        public void OnNext(KeyValuePair<string, object> value)
        {
            if (value.Key.Contains("LazyLoading"))
                Console.WriteLine("Lazy loading with EF detected: " + value.Value);
        }
    }
    ```
    
    - Place a debug breakpoint on the line having the Console.WriteLine, so you can have the complete stacktrace and know where the lazy loading is happening exactly in your code when you invoke the endpoint.

- Maybe using AsNoTracking
- Not calling SaveChanges() in a for loop
- Understand when the linq query is becoming an actual SQL query so you don't end up fetching too much data from the database or worst loading all the table content
- Consider using another library, such as Dapper
- Consider changing the table structure to better suit the request(s) of the app
- Consider doing raw SQL or stored proc
- Profiling SQL Server to get the generated query and investigate
- Profiling SQL requests returning too much data or taking too much time.
    
    Here is an example with EF core and a custom DbCommandInterceptor:

    Register the custom interceptor:
    ```
    public class MyDbContext : DbContext
    {
        protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
        {
            #if DEBUG
            optionsBuilder.AddInterceptors(new MyInterceptor());
            #endif
        }
    }
    ```

    Add the following class:
    ```
    public class MyInterceptor : DbCommandInterceptor
    {
        private string _lastStackTrace;
        
        public override DbDataReader ReaderExecuted(
            DbCommand command,
            CommandExecutedEventData eventData,
            DbDataReader result)
        {
            _lastStackTrace = Environment.StackTrace;
            return result;
        }

        public override object ScalarExecuted(
            DbCommand command,
            CommandExecutedEventData eventData,
            object result)
        {
            _lastStackTrace = Environment.StackTrace;
            return result;
        }

        public override int NonQueryExecuted(
            DbCommand command,
            CommandExecutedEventData eventData,
            int result)
        {
            _lastStackTrace = Environment.StackTrace;
            return result;
        }

        public override ValueTask<DbDataReader> ReaderExecutedAsync(
            DbCommand command,
            CommandExecutedEventData eventData,
            DbDataReader result,
            CancellationToken cancellationToken = default)
        {
            _lastStackTrace = Environment.StackTrace;
            return new ValueTask<DbDataReader>(result);
        }

        public override ValueTask<object> ScalarExecutedAsync(
            DbCommand command,
            CommandExecutedEventData eventData,
            object result,
            CancellationToken cancellationToken = default)
        {
            _lastStackTrace = Environment.StackTrace;
            return new ValueTask<object>(result);
        }

        public override ValueTask<int> NonQueryExecutedAsync(
            DbCommand command,
            CommandExecutedEventData eventData,
            int result,
            CancellationToken cancellationToken = default)
        {
            _lastStackTrace = Environment.StackTrace;
            return new ValueTask<int>(result);
        }

        public override void CommandFailed(
            DbCommand command,
            CommandErrorEventData eventData)
        {
            _lastStackTrace = Environment.StackTrace;
        }

        public override Task CommandFailedAsync(
            DbCommand command,
            CommandErrorEventData eventData,
            CancellationToken cancellationToken = default)
        {
            _lastStackTrace = Environment.StackTrace;
            return Task.CompletedTask;
        }

        public override InterceptionResult DataReaderDisposing(
            DbCommand command,
            DataReaderDisposingEventData eventData,
            InterceptionResult result)
        {
            if (eventData.ReadCount > 1000 || eventData.Duration > TimeSpan.FromMilliseconds(500))
            {
                Console.WriteLine("The SQL request returned too much data or took too much time."
                                + Environment.NewLine
                                + "ReadCount: " + eventData.ReadCount
                                + Environment.NewLine
                                + "Duration: " + eventData.Duration
                                + Environment.NewLine
                                + "StackTrace:"
                                + Environment.NewLine
                                + Environment.NewLine
                                + _lastStackTrace
                                + Environment.NewLine
                                + Environment.NewLine);
            }
            return result;
        }
    }
    ```

    It will log to the console if the SQL request end up returning more than 1000 entries or took more than 500 ms. In all case, you should probably investigate.

## SQL Server
- SQL Index:
    - order of columns matter
    - maybe creating indexes on columns used in "WHERE", "ORDER BY", "GROUP BY" and "DISTINCT" clauses
    - review existing indexes to remove unecessary one
    - avoid using clustered primary key on not sequential unique identifier (aka guid) to avoid high index fragmentation. Consider using a sequential guid or a non-clustered index for the primary key
- review query plan to avoid table/index scans
- Do not use function in WHERE Clauses since it will prevent using index.
- SQL collation do matter if we have to use COLLATE operation on a property and that prevents us from being able to use an index
- Text comparisons are based by default on the collation of the database. The default collation for English (en-US) systems is SQL_Latin1_General_CP1_CI_AS. CI = case insensitive and AS = accent sensitive. So, it would not be necessary to transform to upper or lower case before comparing strings.
- Ask yourself how the app will evolve in terms of quantity of data in the tables (it helps to prioritize the efforts to be made)
- Set up a mechanism so as not to have to keep all the data (deletion/archiving after a while)
