# Report for assignment 4


## Project

Name: OpenRefine

URL: https://github.com/OpenRefine/OpenRefine

Description: OpenRefine is a tool for data-wrangling i.e. a tool for cleaning and transforming messy data. 

## Onboarding experience

Previously we worked with Apache Commons Imaging but we decided to change to a new project as we thought it was quite hard to work with images. Furthermore, we didn’t particularly like their issue-handling system, although we didn’t spend too much time looking into it.

The onboarding experience of OpenRefine was overall very good. The page describing how to build, test, and run has very clear instructions. All necessary prerequisites are listed and the site explains everything step by step in a very nice way. It has guides separate guides for different operating systems and IDEs which makes it very easy to get started. The commands for building the project (`./refine build`) and testing (`./refine test`) are also simple which makes up for a nice experience. We didn’t encounter any problems during the setup of the project which was very nice. 

However, what is not explained in the onboarding documentation is that the database tests are not executed locally with the command `./refine test`. This became a problem for us when we submitted our pull request as we failed the CI-server tests even though we passed all tests locally. There are no instructions on how to trigger these tests which was very frustrating as we were unable to troubleshoot why the tests on the CI-server failed. This bug ctually resulted in a new pull request by Tom Morris which was later merged into the master branch.

If we compare the onboarding experience of Apache Commons Imaging and OpenRefine, we would say that OpenRefine had a slightly better onboarding experience. Both projects were built successfully without having to install anything except Maven, but OpenRefines onboarding documentation was a bit more beginner-friendly. For example, Apache Commons Imaging didn’t explain that you had to run `mvn build` and `mvn test`, presumably since they thought it was obvious. 

## Effort spent

### Elias:

Total time spent: 25h

#### Time log:
##### 26/2/2024: 3h
13-14: 1-hour meeting with everyone: reading and understanding the assignment and looking for a project to work with.

20:30-22:30: 2 hours individually looking for a suitable issue to work with. Looked at many different repositories such as SpringFramework and ElasticSearch.The goal was to find an issue related to refactoring with sufficiently high complexity. Initiated a discussion about this issue in the Spring Framework https://github.com/spring-projects/spring-framework/issues/32278 but it was too big according to the creator of it. Eventually found an issue at OpenRefine https://github.com/OpenRefine/OpenRefine/issues/6391
which I thought was reasonable but it was too big again. However, after a conversation with the author, he gave us a new issue as a part of it which we decided to work on.

##### 29/2/2024: 6h 10 min
9:15-9:45 Read the assignment and the issue carefully again to fully understand what has to be done.
9:45-10:15 Read up on the contributions guide.
10:15-12:15 Working on issues 2 and 11.  Read the documentation on how to use getParamaterMap and deprecate methods in Java.
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

#### Summary approximation Elias (based on the time log above):
1. plenary discussions/meetings: 1h
2. discussions within parts of the group; 0h
3. reading documentation 3h 30 min; 
4. configuration and setup 2h;
5. analyzing code/output 3h;
6. writing documentation 6h 30minh;
7. writing code 7h 30min;
8. running code 1h 30min


## Overview of issue(s) and work done.

#### Title: 
Remove use of java.util.Properties for URL query parsing

#### URL: 
https://github.com/OpenRefine/OpenRefine/issues/6403

#### Summary: 
Deprecate old homegrown parsing methods of URLs and replace them with a new parsing method implementing `getParamatersMap()`. The new method should return `Map<String, String>` instead of Properties.

#### Scope (functionality and code affected): 
The proposed changes affect 10 different classes and 27 different methods. The functionality is almost the same as before, however, `java.util.Properties` is backed by a hashtable which means it doesn’t allow null values and it is always synchronized and therefore slow. The use of a `java.util.HashMap` fixes these problems

### Requirements 
1. Deprecate all methods in `ParsingUtilities` which return `java.util.Properties`
2. Create a replacement for `static public Properties parseUrlParameters(HttpServletRequest request)` with the signature `static public Map<String, String> parseParameters(HttpServletRequest request)` which uses available functionality for its implementation like `request.getParameterMap()`
3. Modify all uses of `parseUrlParameters` to use the new method and adjust all necessary code to accommodate the new return type
4. Preserve public APIs

Optional (point 3): trace tests to requirements.

## Code changes

### Patch
Link to clean patch (point 4): https://github.com/OpenRefine/OpenRefine/pull/6407/files
PR Considered for acceptance (point 5): https://github.com/OpenRefine/OpenRefine/pull/6407


## Test results

Test logs: 

Path: doc/test-resuts

Test log before: “initialTestResults.html”
Test log after:  "finalTests.html” 


## UML class diagram and its description

Before: doc/UML-diagrams/UMLBefore.jpg

After: doc/UML-diagrams/UMLAfter.jpg

### Key changes/classes affected

The following public methods have been added:
- `parseParamters(HttpServletRequest)` in `ParsingUtilities` 
- `loadDataAndPrepareJob(HttpServletRequest, HttpServletResponse, Map<String, String>, ImportingJob, ObjectNode)` in `ImportingUtilities` 
- `retrieveContentFromPostRequest(HttpServletRequest, Map<String, String>, File, ObjectNode, Progress)` in `ImportingUtilities`

Furthermore, all private methods now take `Map<String, String>` instead of `Properties`


### Relation to design- and refactoring patterns (point 2) :

The "extracting methods" refactoring pattern was applied within ImportingUtilities. In order to avoid code duplication, the code from both `retrieveContentFromPostRequest` and `loadDataAndPrepareJob` was extracted and incorporated into two newly created methods having the same names but taking `Map<String, String>` instead of `Properties`. To do this another helper method was created `propsToMap` which converts `Properties` to a  `Map<String, String>`. By creating a private helper method we avoided code duplication as this functionality was needed in both methods.  Then the old methods call the newly created functions. This way we achieved reusable code with high maintainability. 

Furthermore, getting rid of the homegrown solution of URL parsing and using the more modern approach with `getParametsMap()` is a way to increase the understandability and readability of the code. The previous approach with three different methods was not elegant when the same functionality could be achieved with one line of code. Homegrown solutions are also more error-prone, and therefore this replacement increases the maintainability of the code.

Deprecating methods were also utilized and can be seen as maintainability pattern to signal that certain functionality is being phased out. By deprecating methods, you can preserve public APIs but still signal that there newer versions that should be used instead. 

## Overall experience

### What are your main take-aways from this project? What did you learn?

One key takeaway is that the open-source community is very friendly and helpful, at least in our project. We also learned that it can be somewhat hard to find a suitable issue to work on when you have limited experience with the project. When you have limited knowledge it is hard to understand the requirements in the issues and therefore also implement a good solution. We also learned about git rebasing which we did not have experience with before.

### How did you grow as a team, using the Essence standard to evaluate yourself?

The team's progress has been minimal throughout the course, and we find ourselves still in the "formed" stage as per the essence standard, with little development to show. We have very different styles of working, where Elias prefers to do things early in the project, whereas the other team members prefer to do the tasks very close to the deadlines of the task. Consequently, the team is not working as one cohesive unit. The team's failure to be done with the assignments to the presentation deadlines over and over again leads to mistrust within the group which prevents us from taking the next step “Collaborating” where trust is a requirement. On the next level “Performing”, there is a requirement that “The team consistently meets its commitments“, which we also fail since we always miss deadlines. What has been lacking from the team is a good solid project plan from the beginning of each task. With much better project management and planning a lot of irritation and stress could have been eliminated. Communication has somewhat improved over the course, but there is still work to do. 

### SEMAT Kernel - Requirements (point 6):  						

The first state of the requirements is “conceived”. Antonin Delpeuch (wetneb), the top contributor of OpenRefine, Tom Morris (tfmorris), ranked fourth, and Thad Guidry, ranked eighth, discussed and agreed upon the need for getting rid of `java.util.Properties` in issue 6391. As top contributors to the project, they can be seen as key stakeholders.  Another key stakeholder of the project is the users. There is no information if the users have required the support for null values but it is obvious that they would benefit from a faster system which the new system will give them. From this, we can conclude that the requirements were conceived by the stakeholders. 

The next state is “bounded”. Tom Morris bounded the requirements by setting up multiple sub-issues to the big problem they wanted to address. We addressed one of these sub-issues (6403) and the requirements for it were very clear and bounded. For example, it was clearly stated that this particular sub-issue was bound to address the use of Properties for URL query parsing and not anything else. 

The requirements are also “coherent”. The stakeholders want to get rid of `java.util.Properties` in the whole system and not only a part of it to have a coherent system. 

Furthermore, the requirements are “acceptable”. There are no real drawbacks to the proposed solution, as it preserves public APIs and results in a faster system with better functionality as it will now support null values. Key stakeholders seemed to have agreed on the requirements, although some alternatives were discussed such as Java Records.

After “acceptable” comes “addressed”. Throughout the development, we kept an open discussion with Tom Morris (key stakeholder) about our implementation to make sure that it satisfied the requirements. He approved our implementation but also gave tips on how to further improve it. All of the listed requirements in the issue were addressed and approved by Tom Morris which means that the requirements were also “fulfilled”.

Looking at our issue from a broader perspective, a drawback and limitation of our work is that it doesn’t seem to have been a feature demanded by the users, a key stakeholder. Therefore you could argue that our work doesn’t provide much value. Users may have wanted us prioritize to implementing other features, which would have benefited them more. The absence of any discussion with this key stakeholder raises concerns about the widespread acceptance of the requirements. The lack of engagement with all relevant stakeholders suggests a potential oversight in ensuring unanimous approval. However, many small unnoticeable improvements like this will in the end benefit the end user. On a positive note as well, top-contributing developers seemed to be happy about our work they approved that we had satisfied the requirements which means they think we did valuable work. 




#### Optional (point 7): 
Is there something special you want to mention here?

#### Optional (point 8):
 In the context of Jonas Öberg's lecture last week, where do you put the project that you have chosen in an ecosystem of open-source and closed-source software? Is your project (as it is now) something that has replaced or can replace similar proprietary software? Why (not)?


## Optional (point 1): Architectural overview.


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

