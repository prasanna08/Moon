# Google Summer of Code’17 with Oppia

I’m Prasanna Patil. I’m a computer science undergraduate, currently in my senior year, from India. I have been contributing to open source for last one and a half years.

Google Summer of Code is an open source initiative of Google where students get to work on open source projects during summer and get paid for it. I was selected by Oppia for GSoC, 17 for working on a machine learning based feedback system.

## About the project

Oppia is an online community of tutors and learners. It provides platform to create interactive lessons on variety of topics. The lessons are called explorations in Oppia. Explorations have many kinds of interactions to interact with students. One such interaction is code interaction where students write python program in an editor for a problem stated by the tutor. Oppia provides feedback for the submitted program and if the program is correct then the student is allowed to proceed further in the exploration.

Oppia provided these feedbacks based on hard rules. These hard rules can be defined by tutor while creating an exploration. Hard rules are very stringent and provides feedback based on output of the program rather than looking at what program actually is. Therefore the requirement was to build a service for providing more dynamic feedback based on the program text rather than the program output.

The project “Machine learning for Code interaction” aims to provide feedbacks to students’ program dynamically by using machine learning methods. Although there was initial infrastructure available for machine learning, there were many changes to be made to improve entire functionality. This structure was never released publicly because it wasn’t complete yet.

Initially Oppia used to train machine learning models on the server itself using self-implemented algorithms. Oppia uses Google App Engine to build its server. Google App Engine (as of this project) doesn’t support any machine learning libraries widely used today. Therefore we had 2 choice here: (1) Implement algorithms ourselves and (2) Use separate virtual machine to provide ML as a service.

We decided to go with 2nd choice. The reason for this is that self implemented algorithms were inefficient and not as accurate as the ones found in machine learning libraries. Also use of existing machine learning libraries increases long term maintainability and reduces scope of errors in system. The use of VM allows us to build and implement many complex machine learning algorithms without any restrictions as opposed to GAE which has many restrictions. The VM should act as Machine-learning-as-a-Service.

However with this idea an internet connection is always required for doing prediction for new input answer. Moreover since prediction was to be done in real time, we can’t rely on VM for prediction which could have a high latency. We had two choices for prediction API: (1) Perform prediction on GAE (2) Perform prediction on learner’s machine. We decided to go with frontend prediction where we replicate the prediction APIs of machine learning algorithm in Javascript which uses the trained classifier’s data, referred as classifier-data, from Oppia-ml. This also makes the prediction API completely offline, in learner’s machine.

Therefore the aim of the project was to build a new infrastructure for machine learning in Oppia (which we refer to as Oppia-ml) and implement a classifier for python programs. In summary, the project can be divided in 2 parts:
1. Build infrastructure for training classifier models using machine learning algorithm. The goal was to create an infrastructure using a separate Virtual Machine server where machine learning algorithms can be trained efficiently. The training should happen outside Oppia and without having tight coupling between training server and Oppia.
2. Research and implement a machine learning classifier for python programs. The goal was to go through existing research papers and find out a solution that can be used in Oppia for providing feedback to code interaction.

## Final work

The aim of the project was to set up infrastructure for training machine learning algorithms outside Oppia. This was achieved with the help of Oppia-ml. Oppia-ml is a separate code executing on a VM. The main purpose of having Oppia-ml is to be able to use machine learning libraries and train machine learning models without degrading performance of Oppia.

### Oppia-ml architecture

There are three main parts in the architecture shown above:

![Markdowm Image][1]

1. Frontend: this is where the student and the tutor interacts with Oppia. The prediction for new answers submitted by students also happens in frontend.
    * Initially, the creator (tutor) of an exploration creates and submits the training data in the editor view of the exploration. Then this training data is transferred to the backend along with other modifications made by creator. A new classifier training job is created whenever creator creates the training data for the first time or modifies the training data.
    * The contents of exploration along with the classifier-data is transferred to the frontend when playing an exploration. The classifier-data contain enough information to perform prediction in frontend. When student submits a new answer, it goes through hard rules for classification and if hard rules fail to classify the answer then the answer is submitted to classifier prediction service for prediction using the classifier-data.

2. Backend: this is where the database are stored and many other services are provided to frontend. It also exposes services for communication with Oppia-ml.
    * Backend mainly stores 3 things which are relevant to this project. First is classifier training job. Second is trained classifier data. The classifier training job is created whenever creator modifies training data of an exploration in frontend. The trained classifier data stores the extracted classifier data which can be utilized for prediction in frontend.

3. Oppia-ml: this is where actual training takes place.
    * Oppia-ml is a separate code which executes outside the Oppia, on a separate VM instance. Oppia-ml keeps polling the Oppia for new training jobs and trains machine learning classifier for them. Once the classifier is trained, it extracts the trained model data from classifier and sends it back to the Oppia where it is stored in database.

The second part of the project was to research and implement classifier for code interaction. During my research I came across many interesting papers for code plagiarism detection, algorithm detection, program similarity ranking. A simple SVM classifier with BoW vector was used as a baseline. I experimented with many standard algorithms such as SVM, Naive Bayes, Decision Tree, Random Forest, Multilayer perceptron etc..
All of these algorithms were trained using various preprocessing steps and compared using their F1-score, accuracy (on validation set). In the end, an ensemble of Winnowing and KNN and SVM classifier was selected for code classification. Links to the relevant documents, dataset, code and spreadsheets are available in references sections.

The code classifier and its prediction service in frontend have been implemented successfully. We are currently going through end to end testing of entire system. Once that is done the feature will be initially released for developers and then to the production in  succession.


