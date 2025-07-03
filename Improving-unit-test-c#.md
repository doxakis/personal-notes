# Ideas to improve unit test in c#

- reduce the amount of code in a given method
- reduce the complexity of the code in a given method
- consider splitting code in multiple layers (controllers, services, utils, repositories, background tasks, factories, etc)
- consider using design pattern where it make sense
- finding the right balance between repeating similar line of code and creating generic code. Generic code may be more complex to maintain and repeating similar line of code reduce code complexity, generate more code to maintain, allow deliverying faster code to production (so, reducing time to market) and maybe less coordination between different teams.
- prefer using immutable data structure to reduce side-effect in other methods
- do not use static method or if required, isolate them in utils with an interface. So, it can be mock
- do not use extension method. (it act as a static method)
- use factory pattern for object creation if you want to be able to mock the creation operation
- use nuget package EnvironmentAbstractions to be able to mock System.Environment
- use nuget package System.IO.Abstractions to be able to mock file system operation on files and directory
- consider using TimeProvider to be able to mock date and time
- consider using nuget package NSubstitute for mocking object
- use [ExcludeFromCodeCoverage] to exclude code from code coverage if you have a good reason. e.g. Program.cs, config files, utils allowing for mocking methods
- consider using project like https://github.com/danielpalme/ReportGenerator for having visualizing code coverage in your projects
