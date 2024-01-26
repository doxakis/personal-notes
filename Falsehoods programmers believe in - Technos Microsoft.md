# Falsehood programmers believe in - Technos Microsoft

### Entity Framework, c#, DateTime et SQL Server
Comme DateTime à un "Kind" pour dire si c'est Local ou Utc, ce n'est pas obligatoire d'utiliser DateTimeOffset. Toutefois, il est important de savoir que si l'on conserve une date comme datetime ou datetime2, le "Kind" n'est pas conservé en BD. Donc, lorsqu'on va lire une date de la BD. Le "Kind" sera "unspecified". Certains ont tenté des appproches pour fixer ce comportement (ref: https://github.com/dotnet/efcore/issues/4711)

### Azure + c# + DateTime.Now
On pourrait croire que le timezone d'une instance d'App Service est en fonction du lieu du centre de données. En pratique, c'est en UTC. DateTime.Now donne donc l'heure en UTC. Idéalement, on ne devrait pas modifier le timezone du serveur, mais il est possible de le modifier en utilisant la configuration WEBSITE_TIME_ZONE.

## c# + HttpClient
Même si la classe HttpClient est disposable, il ne faut pas disposer car le socket associé n'est pas immédiatement libéré. Disposer de nombreuses instances de HttpClient peut amener à manquer de socket et ralentir une application web. La classe HttpClient est thread safe et il pourrait être tentant de l'assigner à un champs d'instance static en vue de le réutiliser. À ce moment-là, on fixe le problème, mais on en créé un autre. Il ne tient plus compte des changements au niveau DNS. L'une des solutions est d'utiliser IHttpClientFactory. (ref: https://docs.microsoft.com/en-us/dotnet/architecture/microservices/implement-resilient-applications/use-httpclientfactory-to-implement-resilient-http-requests)

## SQL Server

Si on ne met pas de order by, il ne va pas trier forcément en fonction de la primary key.

### Type datetime

Il est bon à savoir qu'il offre une précision limitée. Ce n'est pas précis à la milliseconde près. C'est possible de remarquer une différence de 3-4 millisecondes. Si c'est un enjeu, il existe des types offrant plus de précision: datetime2 ou datetimeoffset pour conserver aussi le timezone.

### Backup sous le format bacpac

Via le portail Azure sur une resource SQL database, il est possible d'exporter la BD dans un format bacpac et ce sans avertissement même sur une BD en cours d'utilisation. Le format bacpac ne peut pas être utilisé comme backup d'une BD en cours d'utilisation directement, car il ne respecte pas les transactions SQL et un restore peut échouer en causant des violations de foreign key par exemple. Ça démeure pratique pour déplacer une BD d'un serveur à l'autre ou pour passer au cloud, car c'est un format de données très compressé. Si vous souhaitez tout de même utiliser ce format sur une BD en cours d'utilisation. Vous pourriez faire une copie de la BD et ensuite générer le bacpac avec la copie.

### Collation

Les comparaisons de texte se base par défaut sur la collation de la BD. La collation par défaut pour les systèmes en anglais (en-US) est SQL_Latin1_General_CP1_CI_AS. CI = case insensitive et AS = accent sensitive. Donc, il ne serait pas nécessaire de transformer en majuscule ou miniscule avant de comparer des chaînes de caractères.

