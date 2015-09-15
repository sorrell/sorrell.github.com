---
layout: post
title: Getting SSIS LineageIDs
tags: [SSIS, BIML]
---
# {{ page.title }}

When I saw the output Error columns in SSIS's Dataflow Task (DFT) for the first time, I thought I had surely misconfigured something, or not followed a certain step.  I mean, I was working with actual data and columns that had human-readable names - but I was getting integers in these Error Code/Column columns...

But like I said, I figured it was operator error, and so I searched my way into hours of disappointment. Microsoft *was* kind enough to enhance the error description, [as described here](https://msdn.microsoft.com/en-us/library/ms345163.aspx).  But that still didn't get a column *name*.

When it came to getting the column name, all signs pointed back to that LineageID - it is the link between a column name at a certain point in the pipeline, and the integer SSIS is using to identify that column.  In the existing answer set on how to grab LineageIDs, I found a web of hacks that not only didn't work, but seemed to be pointing in opposite directions (are my LineageIDs in my dtsx, and are they `ints` or text?).  So here are a couple of important takeaways if you deviate from the solution I will provide later.

1. If you are using SSIS 2012 (Denali) and beyond, then any solution that attempts to get `LineageIDs` by reading the dtsx or trying a `Convert.ToIntXX` call will end in disappointment because LineageIDs in the dtsx are now stored as textual path references.
2. Grabbing the LineageIDs in the DFT can be a little tricky.  When you are sitting on the bottom of the flow, you lose visibility of columns up the chain (see [this excellent post](http://stackoverflow.com/a/20352981/974077) for a great description).  Due to R/W permissions on variables, and that solution is not explored here, but you could conceivably grab a reference to your `Package.Executables`, and explore them later in the DFT for some Just-In-Time LineageID lookup fun (you'll also likely need a reference to the ManagedDTS assembly).

## Solution
I realize that parts of this solution may not work for everyone, say those people with a parent package with a million child packages, and a billion columns.  But it's a proof of concept, and I'm sure you'll find a way to make this work at runtime in your own situation.  A lot of credit here goes to the method outlined [by Dougbert in this post](http://dougbert.com/blog/post/Adding-the-error-column-name-to-an-error-output.aspx).

Here's the basic algorithm we'll follow in the code.

1. Create a `Dictionary<int, string>` variable (let's call it `lineageIds`) to store our LineageIDs
1. Create a Script Task ((SCT) GetLineageIDs) in the Control Flow  
  a. Grab the in-memory package Executables via `lineageIds`'s Parent  
  a. Iterate the package's `Executables` searching for `Dataflows` to extract the LineageIDs
1. Create a Script Component ((SC) GetErrorColNameDesc) to perform lookups on the `lineageIds` Dictionary to extract both the name of the error column and error description

![Overview](/images/201509-SSIS/Overview.png)

### (SCT) GetLineageIDs  
The Script Task to get the LineageIDs is straight forward. There is a section of code writing out to a temp file to show the proof of concept. Just follow these steps before implementing the code.

1. Give the Script Task ReadWrite access to `User::execsObj` (your Executables object) and `User::lineageIds` (your `Dictionary<int,string>`).
1. Add an assembly reference to `Microsoft.SqlServer.DTSPipelineWrap.dll` for the `MainPipe` object
1. The code below is copied from my BIML implementation, so you may only want to copy the methods and namespace references, and use your own/already exists class declaration.

Now for the code.

```c#
using System;
using System.Data;
using Microsoft.SqlServer.Dts.Runtime;
using Microsoft.SqlServer.Dts.Pipeline.Wrapper;
using System.Windows.Forms;
using System.Collections.Generic;
using System.IO;

[Microsoft.SqlServer.Dts.Tasks.ScriptTask.SSISScriptTaskEntryPointAttribute]
public partial class ScriptMain : Microsoft.SqlServer.Dts.Tasks.ScriptTask.VSTARTScriptObjectModelBase
{

  Dictionary<int, string> lineageIds = null;

  public void Main()
  {
      // Grab the executables so we have to something to iterate over, and initialize our lineageIDs list
      // Why the executables?  Well, SSIS won't let us store a reference to the Package itself...
      Dts.Variables["User::execsObj"].Value = ((Package)Dts.Variables["User::execsObj"].Parent).Executables;
      Dts.Variables["User::lineageIds"].Value = new Dictionary<int, string>();
      lineageIds = (Dictionary<int, string>)Dts.Variables["User::lineageIds"].Value;
      Executables execs = (Executables)Dts.Variables["User::execsObj"].Value;

      ReadExecutables(execs);

      // Just proof of concept to see the results before you dedicate your time to the solution
      // Delete this code in your actual implementation
      using (StreamWriter writetext = new StreamWriter(@"C:\temp\write.txt", true))
      {
          foreach (var kvp in lineageIds)
              writetext.WriteLine(kvp.Key + " : " + kvp.Value);
      }
      Dts.TaskResult = (int)ScriptResults.Success;
  }

  private void ReadExecutables(Executables executables)
  {
      foreach (Executable pkgExecutable in executables)
      {
          if (object.ReferenceEquals(pkgExecutable.GetType(), typeof(Microsoft.SqlServer.Dts.Runtime.TaskHost)))
          {
              TaskHost pkgExecTaskHost = (TaskHost)pkgExecutable;
              if (pkgExecTaskHost.CreationName.StartsWith("SSIS.Pipeline"))
              {
                  ProcessDataFlowTask(pkgExecTaskHost);
              }
          }
          else if (object.ReferenceEquals(pkgExecutable.GetType(), typeof(Microsoft.SqlServer.Dts.Runtime.ForEachLoop)))
          {
              // Recurse into FELCs
              ReadExecutables(((ForEachLoop)pkgExecutable).Executables);
          }
      }
  }

  private void ProcessDataFlowTask(TaskHost currentDataFlowTask)
  {
      MainPipe currentDataFlow = (MainPipe)currentDataFlowTask.InnerObject;
      foreach (IDTSComponentMetaData100 currentComponent in currentDataFlow.ComponentMetaDataCollection)
      {
          // Get the inputs in the component.
          foreach (IDTSInput100 currentInput in currentComponent.InputCollection)
              foreach (IDTSInputColumn100 currentInputColumn in currentInput.InputColumnCollection)
                  lineageIds.Add(currentInputColumn.ID, currentInputColumn.Name);

          // Get the outputs in the component.
          foreach (IDTSOutput100 currentOutput in currentComponent.OutputCollection)
              foreach (IDTSOutputColumn100 currentoutputColumn in currentOutput.OutputColumnCollection)
                  lineageIds.Add(currentoutputColumn.ID, currentoutputColumn.Name);
      }
  }

  enum ScriptResults
  {
      Success = Microsoft.SqlServer.Dts.Runtime.DTSExecResult.Success,
      Failure = Microsoft.SqlServer.Dts.Runtime.DTSExecResult.Failure
  };
}
```

### (SC) GetErrorColNameDesc
I place this task below my transformations to catch all of the errors through a Union All.  This doesn't require any special assemblies, but does need **ReadOnly access to `lineageIDs`**.

```c#
using System;
using System.Data;
using Microsoft.SqlServer.Dts.Pipeline.Wrapper;
using Microsoft.SqlServer.Dts.Runtime.Wrapper;
using System.Collections.Generic;

[Microsoft.SqlServer.Dts.Pipeline.SSISScriptComponentEntryPointAttribute]
public class ScriptMain : UserComponent
{

  public override void Input0_ProcessInputRow(Input0Buffer Row)
  {
      Dictionary<int, string> lineageIds = (Dictionary<int, string>)Variables.lineageIds;

      int? colNum = Row.ErrorColumn;
      if (colNum.HasValue && (lineageIds != null))
      {
          if (lineageIds.ContainsKey(colNum.Value))
              Row.ErrorColumnName = lineageIds[colNum.Value];

          else
              Row.ErrorColumnName = "Row error";
      }
      Row.ErrorDescription = this.ComponentMetaData.GetErrorDescription(Row.ErrorCode);
  }
}
```
![Overview](/images/201509-SSIS/DFT.png)

### Conclusion
What I like about this solution is I don't have to run any scripts/packages to prepopulate fields - I can just grab the in-memory package Executables, iterate over them, and get what I need.  Again, this is an amalgamation of existing ideas, with most of the inspiration coming from Dougbert.

Here's the nice output I get in SSMS when I need to see where I'm erring!

![Overview](/images/201509-SSIS/Output.png)
