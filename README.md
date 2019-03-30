
## Good practices in development

### The rationale

As software engineers, we believe it is self-evident that the success of most endeavors in the field are dependent on the practices of the team. Choosing the correct tools, the correct architecture and technological solution are important, but our day to day practices, when we don&#39;t move mountains but rather ever so slightly advance our project, make the difference.

Automation projects have overall long feedback loops and success, or failure is measured over vast expanses of time. For this reason, it is important to have a long-term approach when it comes to implementation rather than a quick win view. This has implications for development time initially, but it pays off when it comes to debugging, maintenance, further projects using reusable pieces of code or handoff procedures

It is for these reasons that, regardless of the kind of organization undertaking the implementation, the project should be approached taking into account certain considerations:

1. Scalability – easy introduction of new processes
2. Ability to work in teams – both from an implementation perspective as well as the infrastructure employed
3. Changing landscape (Lack of ownership of the underlying process or software) – it is usually the case that the Automation team will not have ownership of the process that is being automated or over the software/app that is being automated
4. Employee turnover
5. Ability to handle very complex processes without trading off stability of the implementation

Does this feel like we are preaching to the choir? These are the end goals of any organization and these goals and phrases have been used and over-used over time and space.

But we are having this conversation because over the past decade practices have emerged that have constituted the underpinnings for meteoric rises of some organizations involved in technology, companies that did not exist two decades ago and now are market leaders in their respective fields. These practices can be found under different names, but they constitute the same broad category of approaches to developing pieces of software or IT projects. Such practices are usually revolving around creating testing harnesses that fix in place solutions, automated testing frameworks, projects that are always in a state of &quot;incomplete but ready to run&quot; rather than &quot;complete sub-systems and impossible to run&quot;, continuous monitoring of systems and their constituent components and the list can go on.

Test suits are there to ensure that once a solution is found, any future changes will not impact the work completed already. Test harnesses that are run automatically ensure that adding new functionality or alterations in the work done already does not have unintended consequences. Id est, fixes for one case that break other cases if not tested properly.

While there is no book on good practices in RPA (yet), we can infer the translation of such practices.

### General considerations

By their nature, most workflows will be stateful. They rely that the application being automated is in a specific state, on a specific page or screen, with certain controls present or activated or disactivated.

Tests rely on the state of the applications that are being automated, or on the state of the system. They rely on users present in the database, emails in the inbox, documents on a shared drive.

Fail fast and obviously. Failures should be caught in Dev and/or UAT, not in production. **A major fallacy exhibited is the belief that applications have deterministic behaviors**. There are innumerable applications which behave inconsistently, with controls that move, popups that appear and disappear without predictability. For this reason, tests should be performed in loops of tens or hundreds of iterations, for the workflows that are likely to fail.

While automated tests strive to isolate workflows, it is usually the case that more than one functionality is tested. It is not possible to test a hypothetical Search function without testing both &quot;Initialize application&quot; and &quot;Close application&quot;. So, there seems to be an incremental approach to testing, with more advanced tests relying on the good implementation of more &quot;basic&quot; workflows.



### The checklist

### Inside the workflow

1. The workflows should contain, at the minimum, an initial comments/annotations section with:
  1. General purpose of workflow
  2. Arguments (both in and out)
  3. Assumptions: usually regarding the state of the application being automated (what is the current page/screen, what is the current page of the terminal session and so on)
  4. Throws: the list of exceptions that the workflow throws and what is their significance
  5. Output: the state of the system/application/screen/page after the workflow is invoked
2. Windows, UI elements, terminal sessions should be opened/found only once and then passed as arguments to invoked workflows, rather than attach the window at the beginning of workflows. This minimizes the risk of inadvertently attaching a wrong window, especially if there is a possibility of having more than one window of the same process open at the same time
3. Workflows should be atomic in their intended functionality and small enough to express their complete functionality in one sentence or two. If this is not the case, seams should be searched so that the workflow can be further broken up into sub components
4. Workflows that are more complex should be broken up into reusable pieces of code. These complex workflows should strive to leave the overall system in the exact state it was before execution (windows that are open, pages or screens in apps). The state is referring exclusively to the presentation layer, not about data persistence which can obviously be changed. For example: Search for member -\&gt; Open member page -\&gt; extract relevant information -\&gt; Close member page and return to initial landing page
5. When considering the seams that the library/workflows will be divided on, the testing phase should be taken into consideration. There should be a purpose to Unittest the workflow&#39;s functionality
6. Because of the stateful nature of workflows, call between different libraries should be avoided and deferred to invoking workflows for making the decision. For example: library1/workflow1.xaml calling library2/workflow2.xaml. This should be handled via the invoking workflow: process1.xaml invokes library1/workflow1.xaml and after that invokes library2/workflow2.xaml
7. Except for very simple workflows, Dictionaries should be preferred when passing arguments between workflows. The equivalent of an unknown number of arguments until runtime confers flexibility and easier maintenance. Also, it provides for the ability to have the equivalence to &quot;Interfaces&quot;, when different workflows have the same argument &quot;signature&quot; and allow the same invocation (for example in a ForEach loop)

### Outside the workflow

1. Workflows should be grouped by the underlying application they automate
2. Workflows should have meaningful names that convey the exact message of what is their purpose.
3. If workflows are created for later use as libraries, they should not duplicate information. Names such as test\_1 should be used exclusively for dev purposes and removed from the project as soon as they are not needed anymore.
4.
4.Folder structure should be uniform and consistent. At the very least, a basic folder structure should consist of a src (for source) and tests (for tests). Deeper folder structures under src can be used if the workflows can be grouped by logical similarities or functionality. Under tests additional folders can be used to store assets used by the testing framework
 
## Good practices in testing

### The checklist

1. A two-tier architecture should be employed. On the top tier, RunAllTests should handle running all the tests in the libraries. The tests that are run should be library specific.
2. RunAllTests.xaml as it is implemented at the time of the writing of this document, has these features:
  1. Expects a configuration file, usually a csv, which contain a list of testing workflows
  2. Results and summaries are output in a log file as well as an output csv file
  3. Runs all tests in the provided file, using an invoke workflow convention regarding runtime arguments. This is as close as we can come to implementing the concept of &quot;Interface&quot;: multiple workflows adhering to the same calling convention, but with different implementations
3. Individual library tests:
  1. Should conform to calling conventions/contracts with the RunAllTests workflow
  2. Arguments should adhere to the expected input and outputs as:
    1. in\_LogFile (string) containing the path to the log output file
    2. out\_TestsResults (datatable) containing 3 columns named TestCase, Result [PASS, FAIL], Comments
  3. For each test, a new row should be added to the output datatable, containing the test name, result and (optionally) comments
  4. Test workflows, as well as implementation workflows should not be modified with changing landscape. For this reason, configuration files should be used. These files can take multiple formats:
    1. Key-Value mappings, read using ReadCsvAsDict.xaml directly into a String – String dictionary. Possible uses: non-uniform tests (positive: correct\_name, negative: inexistent\_name), common configuration files (usernames, paths, links)
    2. Lists of input values for tests for homogenous tests (the same workflow being tested for both positive and negative outcomes with multiple inputs). Possible uses: search for users (membershipNo, firstName and lastName, non-existing, multiple results for the same inputs), extract information for cases from IWD, check that the information matches