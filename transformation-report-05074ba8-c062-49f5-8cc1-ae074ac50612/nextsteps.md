# Next Steps

## Issues resolved
- Transformed Bookstore.Domain.csproj to net8.0
- Transformed Bookstore.Data.csproj to net8.0
- Transformed Bookstore.Web.csproj to net8.0
- Transformed Bookstore.Cdk.csproj to net8.0
- Transformed Bookstore.Domain.Tests.csproj to net8.0

## Summary

The transformation appears to have completed successfully. No build errors were detected across any of the projects in the solution:

- `Bookstore.Data`
- `Bookstore.Domain.Tests`
- `Bookstore.Cdk`
- `Bookstore.Web`
- `Bookstore.Domain`

The following steps outline how to validate, test, and deploy the migrated solution.

---

## 1. Restore Dependencies

Run a full NuGet package restore to ensure all dependencies are resolved correctly in the new target framework:

```bash
dotnet restore
```

Review the output for any warnings related to package compatibility or deprecated packages that may need to be updated.

---

## 2. Build the Solution

Perform a full solution build to confirm there are no compilation issues:

```bash
dotnet build --configuration Release
```

Address any warnings that surface during the build, particularly those related to nullable reference types or obsolete APIs, as these can indicate areas that may cause runtime issues.

---

## 3. Run Unit Tests

Execute the test project to verify that existing functionality behaves as expected after the migration:

```bash
dotnet test app/Bookstore.Domain.Tests/Bookstore.Domain.Tests.csproj --configuration Release --verbosity normal
```

Review the test results carefully. Any failing tests should be investigated to determine whether they indicate a regression introduced during the migration or a pre-existing issue.

---

## 4. Validate the Data Layer

Since `Bookstore.Data` handles data access, verify the following:

- **Database provider packages** are compatible with the target .NET version. Check the versions of packages such as `Microsoft.EntityFrameworkCore` or any other ORM in use.
- **Connection strings** in configuration files (`appsettings.json`) are correct and accessible in the new environment.
- If Entity Framework Core is used, run a check to confirm migrations are up to date:

```bash
dotnet ef migrations list --project app/Bookstore.Data
```

If migrations are out of sync, apply them:

```bash
dotnet ef database update --project app/Bookstore.Data
```

---

## 5. Validate the Web Project

Run the `Bookstore.Web` project locally to confirm the application starts and behaves correctly:

```bash
dotnet run --project app/Bookstore.Web --configuration Release
```

Check the following:

- The application starts without runtime exceptions.
- All routes and pages load as expected.
- Any middleware or startup configuration (previously in `Startup.cs` or `Program.cs`) has been correctly migrated to the modern minimal hosting model if applicable.
- Static files, authentication, and authorization configurations are functioning correctly.

---

## 6. Validate the CDK Project

If `Bookstore.Cdk` is an AWS Cloud Development Kit project written in .NET, verify that the CDK constructs compile and synthesize correctly:

```bash
dotnet build app/Bookstore.Cdk --configuration Release
```

Then synthesize the CloudFormation template to confirm the infrastructure definitions are valid:

```bash
cdk synth
```

Ensure the AWS CDK CLI version is compatible with the CDK library versions referenced in the project.

---

## 7. Review Target Framework Consistency

Confirm that all projects in the solution are targeting the same .NET version. Open each `.csproj` file and verify the `<TargetFramework>` element is consistent, for example:

```xml
<TargetFramework>net8.0</TargetFramework>
```

Mismatched target frameworks between projects can cause runtime issues even when the build succeeds.

---

## 8. Check for Removed or Changed APIs

Review the code for any usage of APIs that were available in .NET Framework but have changed or been removed in modern .NET. Common areas to check include:

- `System.Web` references, which are not available in modern .NET and should be replaced with `Microsoft.AspNetCore` equivalents.
- `ConfigurationManager`, which should be replaced with `Microsoft.Extensions.Configuration`.
- Any use of `AppDomain` or reflection-based APIs that may behave differently.

The [.NET Upgrade Assistant compatibility analyzer](https://learn.microsoft.com/en-us/dotnet/core/porting/upgrade-assistant-overview) can assist in identifying remaining compatibility concerns.

---

## 9. Deploy to the Target Environment

Once all validation steps pass, deploy the application to the target environment:

```bash
dotnet publish app/Bookstore.Web --configuration Release --output ./publish
```

Copy the contents of the `./publish` directory to the target server or hosting environment and verify the application runs correctly in that environment, including confirming database connectivity and any external service integrations.