# Native AOT for c#
Native AOT is a deployment model that compile ahead-of-time (AOT) to native code.

# Benefits
- Performance improvement (by not having to do JIT compilation in runtime)
- Startup time reduction
- Memory footprint reduction
- Enhanced security

# Inconvenience
- Incompatible libraries or code
- Many limitations: https://learn.microsoft.com/en-us/dotnet/core/deploying/native-aot/?tabs=net7%2Cwindows#limitations-of-native-aot-deployment

e.g. Requires trimming, which has limitations.

- sometime can be a bit slower for some operation.

e.g. System.Linq.Expressions always use their interpreted form, which is slower than run-time generated compiled code.

# Use cases
- serverless functions in order to mitigate cold startup
- it can be used on platforms where JIT, for some reason, is not allowed, such as iOS from Apple.

# How to prepare for it gradually

- Stop using or reduce dependence on reflection or dynamic code
- Use library that are using source generators in order to avoids the need for reflection (e.g. System.Text.Json)
- Enable trimming warnings (`<IsTrimmable>true</IsTrimmable>`)
- Enable partial trimming (`<TrimMode>partial</TrimMode>`) if it is too much work to support it right now and include the specific assemblies that can support it
- Enable trimming everywhere (`<PublishTrimmed>true</PublishTrimmed>`)
- Enable native AOT warnings (`<IsAotCompatible>true</IsAotCompatible>`)
- Enable native AOT (`<PublishAot>true</PublishAot>`)
