# App Dev - Storing Application Data in Cloud Datastore: Node.js
## Objectives
- Harness Cloud Shell as your development environment.
- Integrate Cloud Datastore into a NodeJS application.</br>
![overview](https://user-images.githubusercontent.com/73010204/142084122-9050212a-0e9d-4325-b921-9e2043ea20d2.PNG)
## Activate Google Cloud Shell
Google Cloud Shell is a virtual machine that is loaded with development tools. It offers a persistent 5GB home directory and runs on the Google Cloud. Google Cloud Shell provides command-line access to your GCP resources.</br>
In GCP console, on the top right toolbar, click the Open Cloud Shell button.</br></br>
![a](https://user-images.githubusercontent.com/73010204/141926794-10795137-2194-4980-8e35-c0efc89c2d72.png)</br></br>
Click **Continue**</br>
It takes a few moments to provision and connect to the environment. When you are connected, you are already authenticated, and the project is set to your PROJECT_ID. </br>
**gcloud** is the command-line tool for Google Cloud Platform. It comes pre-installed on Cloud Shell and supports tab-completion.</br>
You can list the active account name with this command:
```sh
gcloud auth list
```
You can list the project ID with this command:
```sh
gcloud config list project
```
## Previewing the Case Study Application
In this section, you access Cloud Shell, clone the git repository that contains the Quiz application, and then run the application.
### Clone source code in Cloud Shell
To clone the repository for the class, execute the following command in Cloud Shell:
```sh
git clone https://github.com/GoogleCloudPlatform/training-data-analyst
```
### Configure and run the case study application
Create a soft link as a shortcut to the working directory:
```sh
ln -s ~/training-data-analyst/courses/developingapps/v1.2/nodejs/datastore ~/datastore
```
Navigate to the directory that contains your working files:
```sh
cd ~/datastore/start
```
Export an environment variable, GCLOUD_PROJECT that references the GCP Project ID:
```sh
export GCLOUD_PROJECT=$DEVSHELL_PROJECT_ID
```
**GCP Project ID in Cloud Shell**</br>
While working in Cloud Shell, you have access to the Project ID in the $DEVSHELL_PROJECT_ID environment variable.</br>
Install the application dependencies:
```sh
npm install
```
The installation may take a couple of minutes.</br>
To run the application, execute the following command:
```sh
npm start
```
### Review the case study application
In Cloud Shell, click Web preview > Preview on port 8080 to preview the quiz application.</br></br>
![1a](https://user-images.githubusercontent.com/73010204/141927676-f2902821-0bf2-43a8-bc79-529817304fac.png)</br></br>
You should see the user interface for the web application like this
![3a](https://user-images.githubusercontent.com/73010204/141927930-7b18c482-ea6b-4f07-8351-8ffc23eca220.png)</br></br>
In the navigation bar, click **Create Question**.
- You should see a simple form that contains textboxes for the question and answers with radio buttons to select the correct answer.
- Quiz authors can add questions in this part of the application.
- This part of the application is written as a server-side web application using the popular Node.js web application framework Express.
![3](https://user-images.githubusercontent.com/73010204/141928163-4f4b4e88-038f-4d6f-be26-9b5580ac0dc6.png)</br></br>

In the navigation bar, click **Take Test**.The client application opens.</br>
![4](https://user-images.githubusercontent.com/73010204/141928281-a1954905-1503-4ab3-9e23-11b87261025f.png)</br></br>
Next, in the navigation bar, click **GCP**.
- You should see a sample (Dummy) question.
- Quiz takers can answer questions in this part of the application.
- This part of the application is written as a client-side web application using the popular JavaScript framework AngularJS.
- It receives JSON data from the server and uses JavaScript in the browser to display questions and collect answers.
![5](https://user-images.githubusercontent.com/73010204/141928479-6ed3204a-f0bf-4775-abde-07420395fda5.png)</br></br>

To return to the server-side application, click Quite Interesting Quiz in the navigation bar.
## Examining the Case Study Application Code
In this section, you use the Cloud Shell text editor to review the case study application code.
### Employ the Cloud Shell code editor
From Cloud Shell, click Open Editor.</br>
![b](https://user-images.githubusercontent.com/73010204/141928788-4e6d7477-f8eb-4c0f-a3c6-403090773524.PNG)</br></br>
Navigate to the **datastore/start/server** folder using the file browser panel on the left side of the editor.</br>
The application is a standard Node.js application written using the popular Express application framework.</br></br>
![7](https://user-images.githubusercontent.com/73010204/141929051-c6c19ad9-df8b-4b48-9c27-db0424bf6dbd.png)</br>
### Review the Express Web application
Select the **datastore/start/server/app.js** file.
- This file contains the entrypoint for the Express application and registers the main application routes for creating questions and delivering quizzes to users.

Select the **datastore/start/server/web-app** folder
- This folder contains the Node.js modules for the server-side web application.

Select the **datastore/start/server/web-app/questions.js** file.
- This file contains the handlers that display the form and collect form data posted by quiz authors in the web application.
 
In the **questions.js** file, find the handler that responds to HTTP POST requests for the /questions/add route.
- Review the source code comments for a detailed description of this handler's functionality.

Select the **datastore/start/server/web-app/views** folder.
- This folder contains templates for the web application user interface using the pug (formerly jade) layout engine.

View the datastore/start/server/web-app/views/questions/add.pug file.
- This file contains the pug template for the Create Question form.Notice how there is a select list to pick a quiz, textboxes where an author can enter the question and answers, and radio buttons to select the correct answer.

Select the **datastore/start/server/api/index.js** file.
- This file contains the handler that sends JSON data to students taking a test. Notice that the handlers also make use of the model object.

Select the **datastore/start/server/gcp/datastore.js** file.
- This is the file where you write Datastore code to save and load quiz questions to and from Cloud Datastore. This module will be imported into the web application and API as the model object for the web application and API

## Adding Entities to Cloud Datastore
Write code to save form data in Cloud Datastore as below
```sh
'use strict';
// TODO: Load the ../config module
const config = require('../config');
// END TODO

// TODO: Load the @google-cloud/datastore module
const {Datastore} = require('@google-cloud/datastore');
// END TODO

// TODO: Create a Datastore client object, ds
const ds = new Datastore({
    projectId: config.get('GCLOUD_PROJECT')
   });
// END TODO

// TODO: Declare a constant named kind
// The Datastore key is the equivalent of a primary key in a // relational database.
// There are two main ways of writing a key:
// 1. Specify the kind, and let Datastore generate a unique 
//    numeric id
// 2. Specify the kind and a unique string id
const kind = 'Question';
// END TODO

// The create({quiz, author, title, answer1, answer2,
// answer3, answer4, correctAnswer}) function uses an
// ECMAScript 2015 destructuring assignment to extract
// properties from the form data passed to the function

function create({ quiz, author, title, answer1, answer2, answer3, answer4, correctAnswer }) {

  // TODO: Remove Placeholder statement
  // return Promise.resolve({});
  // END TODO

  // TODO: Declare the entity key, 
  // with a Datastore generated id
  const key = ds.key(kind);
  // END TODO

  // TODO: Declare the entity object, with the key and data
  const entity = {
    key,
    // The entity's members are represented in a data property.
    // This is an array where each element represents one
    // member in the entity. Each element is an object with a
    // name and a value
    data: [
      { name: 'quiz', value: quiz },
      { name: 'author', value: author },
      { name: 'title', value: title },
      { name: 'answer1', value: answer1 },
      { name: 'answer2', value: answer2 },
      { name: 'answer3', value: answer3 },
      { name: 'answer4', value: answer4 },
      { name: 'correctAnswer', value: correctAnswer },
    ]
  };
  // END TODO

  // TODO: Save the entity, return a promise
  // The ds.save(...) method returns a Promise to the  
  // caller, as it runs asynchronously.
  return ds.save(entity);
  // END TODO
}
```
Save the ...gcp/datastore.js file, and then return to the Cloud Shell command prompt.
## Create an App Engine application to provision Cloud Datastore
Return to Cloud Shell and press Ctrl+C to stop the application.</br>
To create an App Engine application in your project, execute the following command:
```sh
gcloud app create --region "us-central"
```
Note: Although you aren't using App Engine for your web application yet, Cloud Datastore requires you to create an App Engine application in your project.
## Let's run the application and create a Cloud Datastore entity
To start the application, execute the following command:
```sh
npm start
```
![8a](https://user-images.githubusercontent.com/73010204/141930570-9252aa53-51f5-4278-bef4-3031e6020e8e.png)</br></br>
In Cloud Shell, click **Web preview > Preview on port 8080** to preview the quiz application.</br>
Click **Create Question**.Complete the form with the following values, and then click Save.</br>
![10](https://user-images.githubusercontent.com/73010204/141930688-a1b078c5-0209-4e5e-8ce3-8da36d3d5d47.png)</br>
You should be returned to the home page for the application.</br></br>
Go to the Cloud Platform Console and, on the Navigation menu, click Datastore.</br>
![11a](https://user-images.githubusercontent.com/73010204/141930911-2b9335a6-d6bb-42a8-942e-56025653c629.png)</br></br>
**Cogratulation!** The new question is in the Entities list!

## Bonus: Querying Cloud Datastore
### Write code to retrieve Cloud Datastore entities
Return to the code editor. In the **...gcp/datastore.js** file, edit the **function list** as below
```sh
// Lists all questions in a Quiz (defaults to 'gcp').
// Returns a promise
// [START list]
function list(quiz = 'gcp') {
  const q = ds.createQuery([kind])
    .filter('quiz', '=', quiz);
  const p = ds.runQuery(q);
  return p.then(([results, { moreResults, endCursor }]) => {
    const questions = results.map(item => {
      item.id = item[Datastore.KEY].id;
      delete item.correctAnswer;
      return item;
    });
    return {
      questions,
      nextPageToken:
      moreResults != 'NO_MORE_RESULTS' ? endCursor : false
    };
  });
}
// [END list]
```
### Run the application and test the Cloud Datastore query
Save the ...gcp/datastore.js file, and then return to the Cloud Shell command.</br>
Stop the application by pressing Ctrl+C.</br>
Start the application.</br>
```sh
npm start
```
In Cloud Shell, click Web preview > Preview on port 8080 to preview the quiz application.</br>
Replace the querystring at the end of the application's URL with /api/quizzes/gcp.</br></br>
![13](https://user-images.githubusercontent.com/73010204/141936372-71d11003-dff8-439e-b87c-737a46b536e3.png)</br>
Return to the application home page, and click **Take Test**.</br>
Click **GCP**</br>
You should see that the quiz question has been formatted inside the client-side web application!</br></br>
![14](https://user-images.githubusercontent.com/73010204/141936678-aaf59a5a-b1aa-4d5b-acea-40444cd1539c.png)

## Summarize
You 've created Cloud Datastore with Node.js and query data from it

## Reference
https://www.coursera.org/learn/getting-started-app-development/home/welcome
https://googleapis.dev/nodejs/datastore/latest/index.html
