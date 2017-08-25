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

### Weekly progress log
##### Week #1:
* Initial directory setup for Oppia-ml repository. 
* Started working on remote communication facilities on Oppia-ml side. This implements necessary functions for proper communication between Oppia and Oppia-ml.
* Also finished README for new repository. It should act as a guide to install the Oppia-ml locally.
* Pull requests: https://github.com/oppia/oppia-ml/pull/4 (directory setup), https://github.com/oppia/oppia-ml/pull/10 (remote communication functions).

##### Week #2:
* Pull request for remote communication facilities reviewed and merged successfully.
* Finished documentation for Oppia-ml extension. This document describes relation between Oppia and Oppia-ml.
* Finished integrating Oppia-ml repository with Travis-CI.
* Added a new admin config property on Oppia admin page for adding VM security key. This key is used for authenticating communication between Oppia and Oppia-ml by generating signature for messages.
* Pull requests: https://github.com/oppia/oppia-ml/pull/10 (remote communication facilities) https://github.com/oppia/oppia-ml/pull/12 (Travis-CI) https://github.com/oppia/oppia-ml/pull/11 (Few fixes in installation script) https://github.com/oppia/oppia/pull/3542 (Admin config property).

Week #3:
* Started working on main worker process. This is master process on Oppia-ml which coordinates everything on Oppia-ml.
* Added Oppia-ml extension documentation on Oppia’s wiki.
* Started working on design document for MR job to extract training data from production.
* Pull requests: https://github.com/oppia/oppia-ml/pull/15 (main worker process) https://github.com/oppia/oppia-ml/pull/16 (small fix in communication function)
* Wiki documentation: https://github.com/oppia/oppia/wiki/Oppia-ml-Extension

##### Week #4:
* Pull request for main worker process reviewed and merged.
* VM has been completely implemented. Now we can move to deployment testing and make sure that VM code is deployable.
* Started reading research papers for machine learning classifier for code interaction.
* Refining design document for MR job to extract training data from production.
* Summarised my findings on  automatic algorithm recognition based on programming schemas and beacons in research paper doc (link in References section).
* Pull requests: https://github.com/oppia/oppia-ml/pull/15 (main worker process)

##### Week #5:
* Read and summarized assessing roles of variables using program analysis research paper.
* Read and summarized stanford’s moss related research paper. Implemented (and tested with some samples) winnowing algorithm which is used in research paper. 
* Read overcode research paper. Understood the technical details used in this software.
* Started working on implementation of data extraction query controller in a pull request. PR merged successfully.
* Pull requests: https://github.com/oppia/oppia/pull/3581 (data extraction)

##### Week #6:
* Read and summarized the Detecting Source Code Similarity Using Code Abstraction paper.
* Extracted python program dataset from Oppia. Filtered the dataset according to whether program has correct syntax or program compiles successfully and whether program executes (in a restricted environment).
* Started deployment testing of Oppia-ml on a real time GCE instance.
* Fixed issue with GCE metadata platform services.
* Pull requests: https://github.com/oppia/oppia-ml/pull/17 (metadata)

##### Week #7:
* Started tagging the dataset manually. Only programs were available and required to be tagged manually so that performance of various classifier can be compared.
* Tagged a total of 500 programs in the dataset. That should be sufficient to compare classifiers. Programs were classified into one of the 6 classes.
* Started implementing different classifiers (link to the code is present in References section).
* Using SVM + variable name renaming (as preprocessing) for baseline. Baseline performance is ~74% accuracy with 0.74 of F1-score. (link to spreadsheet containing performance of various classifiers is available in References section).

##### Week #8:
* Testing more algorithms such as Decision tree, Random forest, Winnowing + knn, winnowing + svm, winnowing + k-medoids.
* Tested winnowing + KNN + SVM pipeline (ensemble methods). This method provides best F1 score of 0.88 with accuracy of 88%.
* Started working on code classifier on Oppia-ml. The winnowing + KNN + SVM pipeline is selected as code classifier as of now. Submitted PR for code classifier.
* Job request message structure has been changed slightly. Submitted a PR for this changes in Oppia-ml.
* Pull requests: https://github.com/oppia/oppia-ml/pull/18 (job request message structure), https://github.com/oppia/oppia-ml/pull/19 (code classifier)

##### Week #9:
* Code classifier PR is currently under review. More and more work is going on in code classifier.
* Pull requests: https://github.com/oppia/oppia-ml/pull/19 (code classifier)

##### Week #10:
* Code classifier PR is reviewed and merged.
* Started working on frontend prediction API. Since code classifier uses python’s native tokenizer, first task was to translate this python module into equivalent JS code so that it can be used in prediction.
* Implemented necessary preprocessing functions for frontend prediction API, including functions for Winnowing, KNN prediction, SVM prediction.
* Submitted PR for frontend code prediction API for code classifier.
* Pull requests: https://github.com/oppia/oppia/pull/3719 (frontend prediction API for code classifier), https://github.com/oppia/oppia-ml/pull/19 (code classifier)

##### Week #11:
* Implemented all necessary tests for frontend code prediction API. Frontend prediction PR is still under review.
* Pull requests: https://github.com/oppia/oppia/pull/3719 (frontend prediction API for code classifier)

##### Week #12:
* Merged the frontend code prediction API pull request.
* Submitted a PR for dynamic inclusion of JS code required for frontend prediction API. However,this PR was closed because we decided to use old dependencies infrastructure for prediction services because there is some parallel work going on to improve old dependency infrastructure.
* Submitted new PR for enabling code classifier in Oppia.
* Also working on final documentation which serves as a guide to implement your own classifier in Oppia.
* Pull requests: https://github.com/oppia/oppia/pull/3719 (frontend prediction API for code classifier), https://github.com/oppia/oppia/pull/3767 (enabling code classifier).

## Future of project
* In future we are hoping to focus more on user experience and usability of the system. We will be working on modifying creator view of the system so that it is easy to use.
* We are also planning to introduce methods to generate training data quickly. This will encourage more and more creators to use ML facility.
* We are also planning to build a similar classifier for text classification and many other interactions.
* The current version of ML facility provides a solid initial phase of the entire system. In future we are planning to bring more improvements to the system.

## Conclusion
* To conclude, I would like to thank both Google and Oppia for providing me this opportunity. I had very good mentors Anmol and Allan. Also my peers Pranav and Giritheja, who were working on other parts of machine learning system were really helpful. I would also like to thank entire Oppia community for this awesome learning experience.


## References
1. Oppia-ml GitHub repository: https://github.com/oppia/oppia-ml
2. Commits on Oppia-ml: https://github.com/oppia/oppia-ml/pulls?q=is%3Apr+author%3Aprasanna08+is%3Aclosed
3. Commits on Oppia (in span of GSoC’s 3 months): https://github.com/oppia/oppia/pulls?utf8=%E2%9C%93&q=is%3Apr%20author%3Aprasanna08%20is%3Aclosed%20updated%3A%3E%3D2017-03-01%20
4. GSoC project proposal: https://docs.google.com/document/d/1GoiQHKuIoVTLofE5J-rmff3GnpK44T2uylcFKIWLMkk
5. Daily devlog: https://docs.google.com/document/d/1Idpae5ohpdl0I1fGKhJWXTq5mCHiezF1S8qWZKTTaQQ
6. Researching code classifier (summary of various research papers): https://docs.google.com/document/d/1f1A5egdmMUQvAR5a42R8mC-PCU-ZWGq18sFMBuXyefQ
7. Code classifier dataset and code for all experimented classifier: https://github.com/prasanna08/code-classifier-dataset
8. Performance comparison of various classifiers on code classifier dataset: https://docs.google.com/spreadsheets/d/1tUdXqvox6Qbd9gm23JXO8HecEG466v9WL6U4Ek1gSJA

[1]: https://github.com/prasanna08/prasanna08.github.io/blob/master/assets/img/Oppia-ml%20architecture.jpg
