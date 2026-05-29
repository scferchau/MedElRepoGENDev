# Instructions for GitHub CoPilot in the MedEl SCADA project

## General

- Use an informal style, e.g. use the "du Form" in German
- Answer the question short and simple (KISS principle)
- Do not mention the conventions in your responses
- Check existing API signatures in the workspace first and utilize existing methods instead of workarounds
- Do not question the polling/tick architecture of the whole system - it is management-mandated and cannot be changed easily
- TODOs and FIXMEs should be added in the following pattern: `TODO: YYYY-MM-DD/sC: <the-actual-todo-statement>`, e.g. `TODO: 2026-05-11/sC: i am a comment`

## Branching

- Create MVP base branches with the pattern `feature_MSMVP<Number>_{lib|app}/main`
- Create WP feature branches based on the corresponding MVP base branch with the pattern `feature_MSMVP<Number>_{lib|app}/feature_MSWP<Number>_{lib|app}`
- Use `lib` when the repo name ends with `Lib`; otherwise use `app`
- When creating new feature branches, initially create them only locally. A push will occur later manually or targeted to a corresponding new branch on origin.

## Versioning

- When a subbranch for an MVP is created (`feature_MSMVP<Number>_{lib|app}/main`), version numbers should be adapted like this:
	- the minor `<version>` in `Directory.Build.props` should be set to the MVP number, e.g. `0.46.0.0` should turn to `0.46.131.0` for MVP 131.
	- the `ProjectVersionHistory.txt` should get a new entry with `Teams` and `TW` having the default entries, while `Branch: ` is set to the branch for the MVP and `Brief: ` contains the description of the MVP
	- after the changes are done, a new commit message should be created in this fashion: `MOD: increment version to <new-version> for MVP<MVPNr>`
- When merging back the MVP branch to `main` the versions shall be adapted in this way:
	- set the minor version in the `<version>` tag from `Directory.Build.props` to 0 and increment the major version, e.g. `0.46.131.0` turns to `0.47.0.0`
	- the `ProjectVersionHistory.txt` should get a new entry with `Teams` and `TW` having the default entries, while `Branch: ` is set to the main branch and `Brief: ` contains a description like `New main version with merge from <MVP-branch> (<MVP-description>)`

## Compiling

- The solution is divided into different repos (`RepoGENBaseMSLib`, `RepoGENCompMSLib`, `RepoGENModMSLib`, `RepoGENSysLib`, ...)
- All of those repos are compiled separately
- Do *not* compile on repo level to build all projects
- Projects named MSLibBundleGen<Base|Comp|Mod|...> are used to bundle the compilation of all projects of a certain type (Base, Comp, Mod, SysLib, ...)
- Projects GEN<Base|Comp|Mod|...> are used to copy DLL dependencies for the respective type (Base, Comp, Mod, SysLib, ...)
- When adding code on non-test level, compile on project level; if that's not enough, use the MSLibBundleGen<Base|Comp|Mod|...> project
- When adding unit- or integration tests, compile only the respective test project (UnitTestMSLibs or IntegrationTestMSLibs)

## Project references

- To add project references, use this strategy:
	- During development (i.e. while in a branch starting with `feature_`), use project references, e.g.: `<ProjectReference Include="..\..\..\RepoGENBaseMSLib\GENBaseMSLib\MSLibTask\MSLibTask.csproj" />`	
	- Only when merging back to main, project references are gonna be replaced with NuGet package references (will be done manually using a tool)

## C# coding conventions

- Order using statements by type:
	- System using's and other non- (e.g. starting with `System.`) shall be put first
	- Then should follow MSLib using's (starting with `MSLib`)
	- All other using's (e.g. Integration-tests, internal using's) come last
- Use PascalCase for class names, class properties, record names and record properties
- Records have upper PascalCase variable names
- Use camelCase for function names, variables (even constants) and event names (e.g. `myVariableName`)
- Add an underscore for member variables and events, e.g. `private int myPrivateCounter_`, `internal event EventHandler myEvent_`
- Do not ever use `var` to declare variables
- Do not repeat the variable type when initializing, e.g. use `MyClass myCounter = new();`
	- Exception to the above: initialization of interface types where the specific type is needed, e.g. `IMyInterface myInterface = new MyClass();`
- Do not use array initializers with square brackets (`[]`), use regular initialization via the `new` keyword instead (e.g. `int[] arr = new[]{1,2,3};` instead of `int[] arr = [1,2,3];`)
- Add `// === members`, `// === constructors`, `// === public methods`, `// === public properties`, `// === protected methods`, `// === private methods` comments to separate sections in a class; only add those comments if there are members, constructors, methods or properties in the respective section; always add an empty line after the separators
	- Exception: interfaces and single line records do not need these comments
- Declare the namespace as defined until C# 9, e.g.
```Csharp
namespace SampleNamespace
{
// code
}
```
- Avoid hardcoding of string values -> use private const string members instead
- Remove unused using statements
- Classes, enums, records and interfaces shall be in separate files named after the class, enum, record or interface

### Documentation

- Use XML comments for classes and methods
- Always add a `<summary>` tag to describe the purpose of a class or method
- Do not use file-based XML documentation
- For each parameter of a method, add a `<param>` tag

### Testing

- Use MSTest to write unit and integration tests (use `[TestClass]`, `[TestMethod]`, `[TestInitialize]`, `[TestCleanup]`, `[ClassInitialize]`, `[ClassCleanup]`))
- Put unit tests in subprojects UnitTestMSLibs and integration tests in IntegrationTestMSLibs
- Naming convention: 
	- For test classes: prefix "UnitTest" or "IntegrationTest" followed by the name of the class being tested
	- The class name of the unit test shall be a concatenation of `UnitTest`, `<Name of Library-Type>`, `<Class to test>`, e.g.: `UnitTestMSLibLogConsole`
	- If a file with the same name already exists, add the test method there in the public methods section
- Put the unit tests in a folder named after the project being tested, e.g. UnitTestMSLibs/MyProject
- The method names shall be lowerCamelCase and like `<method to be tested>_<context>_<expected result>`, e.g. `dataObjectByName_returnDODataObjectFromDataObjectPool_sameAsExpected`
- The following block shall always be defined in a test class even if the methods are not used:
```Csharp
// === public methods - test framework methods

/// <summary>
/// Set up the class before any test methods are executed.
/// </summary>
/// <param name="testContext">The context information for the test.</param>
[ClassInitialize]
public static void setUpClass(TestContext testContext)
{
    // Class setup code if needed
}

/// <summary>
/// Sets up the test environment before each test method is executed.
/// </summary>
[TestInitialize]
public void setUp()
{
    // Test setup code if needed
}

/// <summary>
/// Executes the cleanup code after each test method is executed.
/// </summary>
[TestCleanup]
public void tearDown()
{
    // Test cleanup code if needed
}

/// <summary>
/// Executes cleanup logic after all the test methods of the test class have been executed.
/// </summary>
[ClassCleanup]
public static void tearDownClass()
{
    // Class cleanup code if needed
}
```
- The actual test methods shall be begin with a block comment follow by a newline: `// === public methods - test methods`	
- Each test should contain commented sections (even if empty) like this:
```Csharp
// --- set up

<setup logic>

// --- execute

<execution of the method(s) under test>

// --- verify

<verification of the execution using asserts>

// --- tear down

<individual tear down logic>
```
- Assert statements shall only be in the `--- verify` block
- To make internals visible to test classes, annotate the mandatory class `AssemblyDescription.cs` in each project, e.g. like that:
```Csharp
[assembly: InternalsVisibleTo("IntegrationTestAMSSysMSApps")]
[assembly: InternalsVisibleTo("UnitTestAMSSysMSApps")]
namespace AMSSysCentralMSAppLib
{
	/// <documentation>
    public class AssemblyDescription : AssemblyDescriptionBase
    {
    	/// <documentation>
        public AssemblyDescription() : base(typeof(AssemblyDescription).Assembly)
        {
        }
	}
}
```

### Code review

- Do a code review of the currently open file (print its name)
- Be pragmatic and concise 
- Focus on critical errors and coding convention errors. 
- Only print errors and print them in a list, sorted by priority top to bottom
- If no errors are found, print a short message that all is fine.

### Logging

- Always use `ILog` loggers, usually available as `logger_` members or `logger()` methods when moving up the class hierarchy
- In test-classes, `ILog` is usually instantiated as a `LogTest` logger: `private ILog logger_ = new LogTest();`
- Logs are written using a `tag_` member which is the name of the class (e.g. `private string tag_ = nameof(MyClass)`)
- Encapsulate variables inside logging with square brackets (e.g. `logger_?.writeLine(LogLevelEnum.debug, tag_, $"Simulated AST State DTO serialized to XML: [{astStateTestXml}]");`)
