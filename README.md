# Power Platform CLI (pac) Code First / Fusion Test
Example of starting with pac cli, then incorporating things that only happen in the service / solution.

Code first elements include:
* PCF Component
* .NET Plugin

Elements autored in the service:
* Table
* Model-driven app
* Custom API

[TODO: Document more detailed repro steps to build from scratch using pac.]

# At a high level, the proces was:

update gitignore to ensure we don't commit files we don't want in the repo
```
# certain unpacked dataverse solution files
**/Controls/*/*
!**/Controls/*/ControlManifest.xml.data.xml
**/PluginAssemblies/**/*.dll

# export-unpack.ps1 puts things in a temp directory ignore in case delete part of script fails
/temp
```

create folders: src/solutions/CodeFirstFusionExample

**pac solution init** in the CodeFirstFusionExample solution folder

rename 'src' subdirectory to 'unpacked-solution' (preference for clarity)

edit CodeFirstFusionExample.cdsproj: replace 'src' text in SolutionRootPath element to 'unpacked-solution'

edit CodeFirstFusionExample.cdsproj: enable SolutionPackageType to Both

**dotnet build** in the CodeFirstFusionExample solution folder

**pac solution import** of the unmanaged zip file in the bin/debug folder

create folders: src/solutions/CodeFirstFusionExample/plugins/CodeFirstFusionExample.Plugins

**pac plugin init**

author plugin in VSCode

**dotnet build** in the CodeFirstFusionExample.Plugins folder

deploy plugin with plugin registration tool

add plugin assembly to deployed unmanaged solution in make.powerapps.com

in powershell set:
```
$solutionName = "CodeFirstFusionExample"
```

Export/Unpack metadata from changes made in make.powerapps.com
```
. .\export-unpack.ps1
```

In CodeFirstFusionExample solution folder run: **pac solution add-reference -p plugins\CodeFirstFusionExample.Plugins**

At this point, the dll does not get copied from within the plugins\CodeFirstFusionExample.Plugins folder to the unpacked-solution/PluginAssemblies folder structure of the unpacked solution.  You can validate this by deleting the dll.  If you do, **dotnet build** will fail. To address this, edit the **CodeFirstFusionExample.cdsproj** file, adding this under the **ItemGroup** with the **ProjectReference** to the **plugin csproj**.
```
<!-- Manually added to ensure plugin dll gets copied before solution pack -->
  <Target Name="CopyPluginAssemblies" AfterTargets="ResolveReferences">
    <Message Text="Copying plugin assemblies..." Importance="high"/>
    <Copy
      SourceFiles="plugins\CodeFirstFusionExample.Plugins\bin\$(Configuration)\$(TargetFramework)\CodeFirstFusionExample.Plugins.dll"
      DestinationFiles="unpacked-solution\PluginAssemblies\CodeFirstFusionExamplePlugins-9732BB82-293C-440C-ADE7-306240D9AAD5\CodeFirstFusionExamplePlugins.dll" />
      <!-- Note that Dataverse doesn't support dll names with '.' in them.  Therefore, destination dll has the '.'' removed -->
  </Target>
```
NOTE: your **DestinationFiles** path will be different because you will have a different GUID

run **dotnet build** in CodeFirstFusionExample solution folder (dll should now get copied into the unpacked-solution/PluginAssemblies folder structure of the unpacked solution )

Added custom api to unmanaged solution using make.powerapps.com to expose plugin logic

Export/Unpack metadata from changes made in make.powerapps.com
```
. .\export-unpack.ps1
```
# BACKUP

**pac solution add-reference** (to plugin)

author plugin in VSCode

run **dotnet build** in plugin folder

deploy plugin with plugin registration tool

manually add plugin assembly to imported solution

**pac solution unpack** (to get plugin metadata)

use plugin registration tool to register messages

pac solution export

pac solution unpack (to get message metadata back in unpacked format)