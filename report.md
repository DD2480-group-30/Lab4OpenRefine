# Report for assignment 4

## Project

Name: OpenRefine

URL: https://github.com/OpenRefine/OpenRefine

Description: OpenRefine is a tool for data-wrangling i.e. a tool for cleaning and transforming messy data.

## Onboarding experience

Previously we worked with Apache Commons Imaging but we decided to change to a new project as we thought it was quite hard to work with images. Furthermore, we didn’t particularly like their issue-handling system, which was on a different website and required an account request to interact with. We didn’t spend too much time looking into it so it may have been doable but instead we decided to switch project with issues on github.

One of our initial choices for a project was spring-framework since it seemed to be active and had many issues to choose from. However, we chose to abandon this due to a few issues including the size of the project and difficulty building it. The size of the project was much larger than the guidelines and also as a result of this it took a very long time to build or run tests. It was also advanced with many different modules and thus a bit difficult to understand. Additionally, Karin and Edvin both experienced issues with their computers freezing and needing a force shutdown while running the tests which took over 15 minutes. We also couldn't find information to troubleshoot these issues easily.This meant that on the tight deadlines of the course it was not feasible to wait so long and deal with crashes.

We then decided to switch to OpenRefine which had a very good onboarding experience. The page describing how to build, test, and run has very clear instructions. All necessary prerequisites are listed and the site explains everything step by step in a very nice way. It has guides separate guides for different operating systems and IDEs which makes it very easy to get started. The commands for building the project (`./refine build`) and testing (`./refine test`) are also simple which makes up for a nice experience. We didn’t encounter any problems during the setup of the project which was very nice.

We only encountered a few issues after that when using the commands starting with `./refine`. One issue was that it was a bit difficult to understand the command line output of the tests since many exceptions (purposeful as a part of the unit test code) were printed directly to the command line. In addition it was not clear where the test results were generated in the repo (this ended up being `main/target/surefire-reports/Refine Unit Tests`). Another issue is that the onboarding documentation did not explain that the database tests are not executed locally with the command `./refine test`. This became a problem for us when we submitted our pull request as we failed the CI-server tests even though we passed all tests locally. There are no instructions on how to trigger these tests which was very frustrating as we were unable to troubleshoot why the tests on the CI-server failed. This bug resulted in a new pull request by Tom Morris which was later merged into the master branch.

If we compare the onboarding experience of Apache Commons Imaging and OpenRefine, we would say that OpenRefine had a slightly better onboarding experience. Both projects were built successfully without having to install anything except Maven, but OpenRefines onboarding documentation was a bit more beginner-friendly. For example, Apache Commons Imaging didn’t explain that you had to run `mvn build` and `mvn test`, presumably since they thought it was obvious. However, both projects were clearly easier to onboard than spring-framework where we dealt with long build times and freezing.

## Group member contributions

More detailed time logs can be found at the end of the report.

|    Member    |    Plenary discussions/meetings    |    Discussions within parts of the group    |    Reading documentation    |    Configuration and setup    |    Analyzing code/output    |    Writing documentation    |    Writing/running code    |    Total time    |
|    :-----    |    :-----    |    :-----    |    :-----    |    :-----    |    :-----    |    :-----    |    :-----    |    :-----    |
|    Edvin    |    2.0 h    |    1.5 h    |    3.25 h    |    2.0 h    |    4.75 h    |    4.0 h    |    3.5 h    |    21.0 h    |
|    Elias    |    1.0 h    |    0.0 h    |    3.5 h    |    2.0 h    |    3.0 h    |    6.5 h    |    9.0 h    |    25.0 h    |
|    Ishaq    |    1.0 h    |    1.5 h    |    5.0 h    |    1.0 h    |    0.5 h    |    8.5 h    |    5.5 h    |    23.0 h    |
|    Karin    |    2.0 h    |    1.25 h    |    2.0 h    |    2.5 h    |    3.0 h    |    4.75 h    |    4.5 h    |    20.0 h    |
|    Total time    |    6.0 h    |    4.25 h    |    13.75 h    |    7.5 h    |    11.25 h    |    23.75 h    |    22.5 h    |    89.0 h    |

## Overview of issue(s) and work done.

### Issue 1

#### Title:
Remove use of java.util.Properties for URL query parsing

#### URL:
https://github.com/OpenRefine/OpenRefine/issues/6403

#### Summary:
Deprecate old homegrown parsing methods of URLs and replace them with a new parsing method implementing `getParamatersMap()`. The new method should return `Map<String, String>` instead of Properties.

#### Scope (functionality and code affected):
The proposed changes affect 10 different classes and 27 different methods. The functionality is almost the same as before, however, `java.util.Properties` is backed by a hashtable which means it doesn’t allow null values and it is always synchronized and therefore slow. The use of a `java.util.HashMap` fixes these problems




#### Requirements
1. **Deprecate methods:**
Deprecate all methods in `ParsingUtilities` which return `java.util.Properties`. This includes the 3 methods with the following headers:
	* `static public Properties parseUrlParameters(HttpServletRequest request)`
	* `static public Properties parseParameters(Properties p, String str) {`
	* `static public Properties parseParameters(String str)`


2. **New parseParameters:**
Create a replacement for `static public Properties parseUrlParameters(HttpServletRequest request)` with the signature `static public Map<String, String> parseParameters(HttpServletRequest request)` which uses available functionality for its implementation like `request.getParameterMap()`
3. **Replace parseParameters:**
Modify all uses of `parseUrlParameters` to use the new method and adjust all necessary code to accommodate the new return type. This needs to be changed in the following files:
	* a. `extensions/database/src/com/google/refine/extension/database/DatabaseImportController.java`

	* b. `extensions/gdata/src/com/google/refine/extension/gdata/GDataImportingController.java`

	* c. `main/src/com/google/refine/commands/Command.java`

	* d. `main/src/com/google/refine/commands/importing/ImportingControllerCommand.java`

	* e. `main/src/com/google/refine/commands/project/CreateProjectCommand.java`

	* f. `main/src/com/google/refine/commands/project/ImportProjectCommand.java`

	* g. `main/src/com/google/refine/importing/DefaultImportingController.java`

	* h. `main/tests/server/src/com/google/refine/importing/ImportingUtilitiesTests.java`

4. **Preserve public APIs:** Make sure that none of the public API's are changed. Add new methods where needed, but don't remove or modify existing ones


### Issue 2

Repo: https://github.com/DD2480-group-30/OpenRefine

#### Title:
Replace Properties with Map<String, String> in Exporters interfaces

#### URL:
https://github.com/OpenRefine/OpenRefine/issues/6403

#### Summary:
Phase out usage of java.util.Properties in Exporter API, in it's StreamExporter and WriterExporter subinterfaces. Each has a single method export() which includes a Properties-typed options parameter in its API. These should be replaced with a Map<String, String>.

#### Scope (functionality and code affected):
The changes in WriterExport would affect 6 different classes with a total of 14 usages. The changes in StreamExporter would affect 2 different classes with a total of 8 usages. Considering the time limit, we only managed to changes in StreamExporter. The functionality havn't change. We implemented a new interface that extends the old one to make ensure backward compatibility. Both implementations of StreamExporter2 export method in (OdsExporter.java and XlsExporter.java) were done. Additionally, two references (CustomizableTabularExporterUtilities.exportRows and ExportRowsCommand.doPost) were made to accommadate the new changes.


### Requirements
1. **New Interfaces:** Introduce new StreamExporter2 and WriterExporter2 interfaces which extends the old interfaces: To distinguish between the old ones and the new one. Add export method that takes Map<String, String> instead of java.util.Properties in the new interfaces: Abstract method that will be overriden in the classes.
2. **Replace in XlsExport:** Implement the new export method in XlsExport.java: The export method is long. Extraction needed to minimize code duplication.
3. **Replace in OdsExport:** Implement the new export method in OdsExport.java to use the new method:
4. **Replace in exportRows:** Modify CustomizableTabularExporterUtilities.exportRows to use Map<String, String>: There is nested functions in the function. Extration needed to minimize code duplication
5. **Replace in ExportRowsCommand** Modify Command.Project.ExportRowsCommand to use the new StreamExporter: getRequestParameters returns Properties, which is used by other class. A new method that return Map<String,String> is required in Command.Project.ExportRowsCommand.

## Code changes

### Patch

Link to clean patch (optional point 4): https://github.com/OpenRefine/OpenRefine/pull/6407/files

PR Considered for acceptance (grading criteria 5): https://github.com/OpenRefine/OpenRefine/pull/6407

## Test results

### Test logs

The results can be found at: [doc/test-results](https://github.com/DD2480-group-30/Lab4OpenRefine/blob/master/doc/test-results)

[View the results in your browser](https://htmlpreview.github.io/?https://github.com/DD2480-group-30/Lab4OpenRefine/blob/master/doc/test-resuts/Summarize.html)

## UML class diagram and its description

Before: [doc/UML-diagrams/UMLBefore.jpg](https://github.com/DD2480-group-30/Lab4OpenRefine/blob/master/doc/UML-diagrams/UMLBefore.jpg)

After: [doc/UML-diagrams/UMLAfter.jpg](https://github.com/DD2480-group-30/Lab4OpenRefine/blob/master/doc/UML-diagrams/UMLAfter.jpg)

### Key changes/classes affected

The following public methods have been added:
- `parseParamters(HttpServletRequest)` in `ParsingUtilities`
- `loadDataAndPrepareJob(HttpServletRequest, HttpServletResponse, Map<String, String>, ImportingJob, ObjectNode)` in `ImportingUtilities`
- `retrieveContentFromPostRequest(HttpServletRequest, Map<String, String>, File, ObjectNode, Progress)` in `ImportingUtilities`

Furthermore, all private methods now take `Map<String, String>` instead of `Properties`

## Overall experience

### What are your main take-aways from this project? What did you learn?

One key takeaway is that the open-source community is very friendly and helpful, at least in our project. We also learned that it can be somewhat hard to find a suitable issue to work on when you have limited experience with the project. When you have limited knowledge it is hard to understand the requirements in the issues and therefore also implement a good solution. This can also cause issues with understanding the amount of work required to solve an issue. We also learned about git rebasing which we did not have experience with before.

### How did you grow as a team, using the Essence standard to evaluate yourself?

The team's progress has been minimal throughout the course, and we find ourselves still in the "formed" stage as per the essence standard, with little development to show. We have very different styles of working, where Elias prefers to do things early in the project, whereas the other team members prefer to do the tasks closer to the deadlines of the task. Consequently, the team is not working as one cohesive unit. The team's failure to be done with the assignments to the presentation deadlines over and over again leads to mistrust within the group which prevents us from taking the next step “Collaborating” where trust is a requirement. On the next level “Performing”, there is a requirement that “The team consistently meets its commitments“, which we also fail since we always miss deadlines. What has been lacking from the team is a good solid project plan from the beginning of each task. With much better project management and planning a lot of irritation and stress could have been eliminated. Communication has somewhat improved over the course, but there is still work to do.

## Optional (point 1): Architectural Overview

### System purpose

The main purpose of OpenRefine is to ease cleaning and transforming data. It is adapted to manage big datasets, which allows users to quickly identify and correct inconsistencies or mistakes. It supports multiple file extensions, like CSV, TSV JSON, XML, and allows for exporting to multiple file extensions as well. Moreover, it allows for connecting with external sources and various web services linking it to datasets to improve data and integration.

### System architecture

The core system is based on a client-server model with the frontend offering a web interface and the backend managing data processing. Communication between client and server is managed through GET and POST AJAX calls. Given the scope and criteria for this overview only the backend architecture will be briefly presented.

### Server-end architecture

OpenRefines server-side architecture uses Java and the Jetty web server and servlet container for its operations with the refineServlet managing client-server interactions. The servlet is in main/src, with the server's core logic being in server/src. This separation allows hosting the servlet in different servlet containers. The architecture supports efficient handling of HTTP requests through AJAX, which enables data exchanges that keep the UI responsive. Command execution such as data cleaning, transformation, and project management is handled by specific classes within the com.google.refine.commands package.

### Projects

The ProjectManager class is responsible for initializing and listing projects. It identifies existing projects by scanning the workspace for their metadata files and organizes them without loading the full data set immediately. These metadata objects created by the ProjectMetadata class include details like project names and modification dates. Projects are loaded into memory only when needed through a lazy loading approach. This enhances the efficiency by only allocating resources to projects’ currently open. The com.google.refine.importing package takes over to manage the actual data loading process once a project is opened.

### Data model

The data model structured around columns, rows, and cells allows for operations like cleaning and historical tracking of changes. The structure is encapsulated within the com.google.refine.model package that defines how data is represented and manipulated. The modifications on the data triggered by user actions are handled through the com.google.refine.model.changes package.

### Backend handling
Changes:  Every modification made to the data is tracked through com.google.refine.history.Change objects. These objects capture the data before and after the modification, which allows for redo and undo operations. This system ensures that all transformations reversible.

History: The history of a project is a collection of these changes. It is organized in com.google.refine.history.History objects. These objects not only keep track of the changes but also categorize them into actions that have been completed and actions that can be undone or redone. This allows users to navigate back and forth through the projects’ transformation history.

Processes: Processes are generated through com.google.refine.process.Process, which are tasks that apply changes on the data. These can be quick actions such as editing cells or larger operations, such as sorting that takes longer to execute. Processes are queued and executed in order.

Operations: Some Processes are based on standard operations by com.google.refine.model.AbstractOperation. These operations are predefined actions that can be applied on the data such as cleaning a column.

In version 4, which is currently under development, it introduces changes to the project history architecture. For more details see documentation

## Optional (point 2): Relation to Design and Refactoring Patterns

The "extracting methods" refactoring pattern was applied within ImportingUtilities. In order to avoid code duplication, the code from both `retrieveContentFromPostRequest` and `loadDataAndPrepareJob` was extracted and incorporated into two newly created methods having the same names but taking `Map<String, String>` instead of `Properties`. To do this another helper method was created `propsToMap` which converts `Properties` to a  `Map<String, String>`. By creating a private helper method we avoided code duplication as this functionality was needed in both methods.  Then the old methods call the newly created functions. This way we achieved reusable code with high maintainability.

Furthermore, getting rid of the homegrown solution of URL parsing and using the more modern approach with `getParameterMap()` is a way to increase the understandability and readability of the code. The previous approach with three different methods was not elegant when the same functionality could be achieved with one line of code. Homegrown solutions are also more error-prone, and therefore this replacement increases the maintainability of the code.

Deprecating methods were also utilized and can be seen as maintainability pattern to signal that certain functionality is being phased out. By deprecating methods, you can preserve public APIs but still signal that there newer versions that should be used instead.

## Optional (point 6): SEMAT Kernel - Requirements      	 

The first state of the requirements is “conceived”. Antonin Delpeuch (wetneb), the top contributor of OpenRefine, Tom Morris (tfmorris), ranked fourth, and Thad Guidry, ranked eighth, discussed and agreed upon the need for getting rid of `java.util.Properties` in issue 6391. As top contributors to the project, they can be seen as key stakeholders.  Another key stakeholder of the project is the users. There is no information if the users have required the support for null values but it is obvious that they would benefit from a faster system which the new system will give them. From this, we can conclude that the requirements were conceived by the stakeholders.

The next state is “bounded”. Tom Morris bounded the requirements by setting up multiple sub-issues to the big problem they wanted to address. We addressed one of these sub-issues (6403) and the requirements for it were very clear and bounded. For example, it was clearly stated that this particular sub-issue was bound to address the use of Properties for URL query parsing and not anything else.

The requirements are also “coherent”. The stakeholders want to get rid of `java.util.Properties` in the whole system and not only a part of it to have a coherent system.

Furthermore, the requirements are “acceptable”. There are no real drawbacks to the proposed solution, as it preserves public APIs and results in a faster system with better functionality as it will now support null values. Key stakeholders seemed to have agreed on the requirements, although some alternatives were discussed such as Java Records.

After “acceptable” comes “addressed”. Throughout the development, we kept an open discussion with Tom Morris (key stakeholder) about our implementation to make sure that it satisfied the requirements. He approved our implementation but also gave tips on how to further improve it. All of the listed requirements in the issue were addressed and approved by Tom Morris which means that the requirements were also “fulfilled”.

Looking at our issue from a broader perspective, a drawback and limitation of our work is that it doesn’t seem to have been a feature demanded by the users, a key stakeholder. Therefore you could argue that our work doesn’t provide much value. Users may have wanted us prioritize to implementing other features, which would have benefited them more. The absence of any discussion with this key stakeholder raises concerns about the widespread acceptance of the requirements. The lack of engagement with all relevant stakeholders suggests a potential oversight in ensuring unanimous approval. However, many small unnoticeable improvements like this will in the end benefit the end user. On a positive note as well, top-contributing developers seemed to be happy about our work they approved that we had satisfied the requirements which means they think we did valuable work.

## Optional (point 8): Open-source Ecosystem

### In the context of Jonas Öberg's lecture last week, where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)?

OpenRefine relies on volunteer contributions and financial support according to the [funding page](https://openrefine.org/funding) of their website. It is backed by a U.S. based nonprofit called Code for Science & Society which takes donations and applies for grants in order to fund a developer, project manager, and necessary development materials. To put the project in the context of the lecture we need to look at its history. It started out as a project called Freebase Gridworks developed by a private company that was made open source in 2010 and could possibly have been one of the “red bricks” which was made into a “green brick” by turning a closed source project into an open source one in order to reduce maintenance requirements and contribute to the community. It was renamed Google Refine after the company was purchased by Google which provided support for the project until late 2012 when it announced it was ending support. After that it became OpenRefine which was fully community-supported and open source. It is something that can be used as a replacement for similar proprietary software where data analysis needs to be done since it has many similar features and robust tools. This is especially true in areas such as science and education where funding may be more restricted than in business. However more businesses, as Jonas Öberg mentioned in his lecture, are opening up to open source software as a base due to the increasing complexity of software and maintenance requirements.

## Time log details

### Elias:

##### 26/2/2024: 3h
13-14: 1-hour meeting with everyone: reading and understanding the assignment and looking for a project to work with.

20:30-22:30: 2 hours individually looking for a suitable issue to work with. Looked at many different repositories such as SpringFramework and ElasticSearch.The goal was to find an issue related to refactoring with sufficiently high complexity. Initiated a discussion about this issue in the Spring Framework https://github.com/spring-projects/spring-framework/issues/32278 but it was too big according to the creator of it. Eventually found an issue at OpenRefine https://github.com/OpenRefine/OpenRefine/issues/6391
which I thought was reasonable but it was too big again. However, after a conversation with the author, he gave us a new issue as a part of it which we decided to work on.

##### 29/2/2024: 6h 10 min
9:15-9:45 Read the assignment and the issue carefully again to fully understand what has to be done.

9:45-10:15 Read up on the contributions guide.

10:15-12:15 Working on issues 2 and 11.  Read the documentation on how to use getParameterMap and deprecate methods in Java.

12:40-15:10 Implementing the issues 3,4,5,6,7,8,9,10,11. Writing pull requests for finished issues. Had trouble with 7, 9, and 10 because of the public API in ImportingUtilities. Not allowed to modify the public methods according to issue.

15:20-19.00: Try to solve the problem with the public API in ImportingUtilities. Created a discussion with tfmoris on how to move forward with it on GitHub. Tried to do a big refactoring of a large function (227 lines of code) but ended up never using the refactoring as I realized afterward it would not be approved in the code review as it was not part of the issue. Furthermore, it didn’t help to solve the public API problem. Do another successful attempt at refactoring where I convert Properties to Map and call a new function with almost the same signature.

##### 1/3/2024: 6 h 30 min
11:20-11:50: Mini-assignment refactoring, including reading the Wikipedia article and watching the video about refactoring.

11:50-13:50: Read the documentation about Java streams and implement a refactoring of propsToMap. Deprecate old public APIs in ImportingUtilities. Read about how you use links when deprecating methods.

14:10-15:25: Improve the before refactoring UML diagram which didn’t include all affected methods and classes by the refactoring. Also didn’t include information about private vs public methods. Create an after-refactoring UML diagram.

15:25-18:10 Read and learn about how to do a git rebase as this was the first time doing it. After some trouble, successfully created a clean branch with a single commit message in order to follow the contribution guidelines. Wrote the pull request at OpenRefine.

##### 2/3/2024: 6h 40 min
9:50 - 10:20 Installed Eclipse and necessary dependencies to build and run the project in order to try to replicate the DataBaseImportingController errors found by the CI-server. Failed to replicate the problem.

10:35-12:30 Install Postgresql, and MySQL to try to replicate the failure with CI-server. A lot of troubles with the installation process and failed to replicate, Wrote a message to OpenRefine in PR on how I should move forward.

12:45-14:35 Write the report - Project description, onboarding experience. Time log.

17:50-19:30 Write the report - Overview of issue(s) and work done. UML class diagram and its description. Relation to design- and refactoring patterns

20:10-20:50 Write the report - Team evaluation (essence standard)

21:25-22:10: Try to solve the DataBaseImportControllerTest problem after some help from Tom Morris

#### 3/3/2024: 2 h 30 min

9:05-10:05 Solved DataBaseImportControllerTest problem, do a rebase and fix a new commit in the PR. Run tests.

11:45 - 13:15. Write the report. SEMAT-kernel (requirements)

### Karin

| Minutes | Description |
|    :-----    |    :-----    |
|    60    |    Group meeting to understand and discuss assignment    |
|    30    |    Look at spring-framework code and browse issues on external site    |
|    60    |    Build spring-framework (address issues with build crashing computer)    |
|    30    |    Browse other projects on github/through articles for ideas    |
|    30    |    Read through issues and discussions for OpenRefine    |
|    30    |    Read through issue #6403 and understand the discussion/necessary work    |
|    30    |    Set up timesheet and record work so far    |
|    60    |    Set up OpenRefine repo (run build and test commands)    |
|    30    |    Find and understand test results and add to repo    |
|    30    |    Code reviews for pull requests    |
|    30    |    Mini refactoring assignment    |
|    30    |    Explore issues (such as #6401) to solve to increase work hours    |
|    15    |    Add timesheet section to essay    |
|    60    |    Work on report: optional point 8    |
|    45    |    Add spring-framework to onboarding documentation and improve essay formatting    |
|    30    |    Group discussion about continuing work    |
|    15    |    Some more report formatting (add URLs)    |
|    30    |    Group discussion about issue #6391    |
|    30    |    Find methods to edit for the issue    |
|    15    |    Discussion on issue page    |
|    60    |    Look at code for issue #6419    |
|    30    |    Brainstorm solutions    |
|    60    |    Code experimentation    |
|    60    |    Update time table to better version    |
|    60    |    Code review/coding #18    |
|    30    |    Discussion with Edvin    |
|    60    |    Test updating 4.0 release branch with our changes    |
|    30    |    Read documentation    |
|    60    |    Coding    |
|    60    |    Follow contributions guide for code    |

### Ishaq

|    Minutes (up to 60)    |    Description    |
|    60    |    Meeting    |
|    60    |    Looking for a project and issues on the list    |
|    60    |    Looking for a project and issues on the list    |
|    60    |    Reading documentation on architecture    |
|    60    |    Reading documentation on architecture    |
|    60    |    Reading documentation on architecture    |
|    60    |    Reading documentation on archictecture and reviewing the code    |
|    60    |    Reading documentation on archictecture and reviewing the code    |
|    60    |    Writing overview of the architecture    |
|    60    |    Writing overview of the architecture    |
|    60    |    Writing overview of the architecture    |
|    30    |    UML class diagram before changes    |
|    30    |    Summarizing test logs into single html    |
|    20    |    Reviewing pull request    |
|    60    |    Discussing issues    |
|    30    |    Discussing issues    |
|    60    |    Started working on new issue #6391 (writer option) before we decided to go with 6419    |
|    30    |    Started working on new issue #6391 (writer option) before we decided to go with 6419    |
|    60    |    Looked into the new issue 6419    |
|    60    |    Writing code/planning for some subissues to 6419    |
|    60    |    Writing code/planning for some subissues to 6419    |
|    60    |    Reviewing pull request     |
|    60    |    Git issues with issues and config    |
|    60    |    Continued writing code    |
|    60    |    Writing code    |
|    30    |    Writing report    |

### Edvin

| Minutes | Description |
| :----- | :----- |
| 60    | First meeting. Reading understanding and discussing the instructions and requirements |
|     |  **Finding and testing setup of projects and looking for reasonable issues:** |
| 30    | spring-boot |
| 30    | c:geo |
| 30    | signalapp |
| 30    | powermock |
| 30    | spoon |
| 30    | spotbugs |
|     | **OpenRefine:** |
| 30    | Set up and run project |
| 30    | Understanding project and project structure |
| 30    | Understanding tests |
| 60    | Understanding the issue we’re working on, which files and lines of code will be affected |
| 30    | Reading and understanding the requirements for the issue |
| 30    | Reading docs of the project (contribution guidelines and other docs for getting a better understanding of the project) |
| 30    | Discussing the forking, setting up of the repo, and repo structure with Karin |
| 60    | Wrote issues #2-#10 |
| 60    | Code review |
| 30    | Reading the doc on test coverage for the project |
| 30    | Inspecting the test coverage for the related code |
| 30    | Time log |
| 60    | Meeting to plan working on the second issue  |
| 30    | Understanding the context of the issue  |
| 60    | Disecting the work and creating issues  |
| 180    | Implement issues  |
| 90    | Code reviews  |
| 90    | Updating the report  |
| 30    | Discussing code  |
