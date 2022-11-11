# Branching strategies
Many various approaches have been proposed for when to create and merge branches, as well as how long to keep them alive.
Each option has its pros and cons.
Because teams and products differ so greatly, there is no one-size-fits-all solution.

The company model is one of the most important decision elements.

There is just one production environment in a Product as a Service business model.
There is usually no reason not to update it frequently.

In product-based models, the product is sent to a large number of clients, and there are several production settings.
Customers will very certainly be utilizing multiple versions of the product, which must be maintained. 

## Trunk-based Development


### How it works?

* A single branch (master) to which all commits are pushed.
* Master deploys to the staging environment. After acceptance testing is done, the updates are sent to production.
* This approach can only function if the automated tests can check every component of the software as a human would. 

### Advantages

* The simplest branching strategy (as there is only one branch).
* Necessitates continous deployment.

### Disadvantages

* Requires feature toggles to conceal under construction features. 
* Major features are especially difficult to integrate since they tend to break practically everything until they are completely finished. 
* Demands sophisticated acceptance testing.

### When to use?

* Works best if the work can be dividied into small features that can be frequently merged with the master.

## Release Branches

### How it works?

* Supportes many versions of the product simulatneously.
* One branch for each release, with numerous tiny hot fix branches branching out from the release branches. 
* Typically, each team is in charge of a distinct release, and updates from one branch are not incorporated into another branch until the entire release is finished. 

### Advantages

* Allows many different versions of the product to live simulateneously.
* Features are not deployed to production till they are complete, hence no need for feature toggles.

### Disadvantages

* Low frequency deployment. 
* No possibility of continuously integrating the changes. 
* Merging fixes is quite complicated. 

## Feature Branches


### How it works?

* A branch for each feature, including hot fix branches.
* The developer works on the feature branch until the feature is finished.
* After the modifications have been tested and approved, they may be deployed to production.
 
### Advantages

* Short delivery cycles.
* Feature-invoking changes that cause widespread system failures are isolated in their own branch until everything is ready and thoroughly tested. 

### Disadvantages

* Teams who use this strategy are more likely to use waterfall methodology.
* Merges may be problematic if the branches live separately for an extended period of time. 

## Forks

Popular among open source communities.

### How it works?

* Rather of cloning the repository, the developer forks it.
* The developer does not have access to the main repository, and merges are only allowed via pull requests. 
* After the pull request has been opened, the people in charge of the main repository review it and decide whether to accept or reject it. 

### Advantages

* Allows total strangers to work on the code.
* Enables entirely asynchronous workflow.
* There is no method for not reviewed changes to be merged with the master.

### Disadvantages

* May be hard to coordinate the team.

## Git flow

### How it works?

* Main, dev, release, features and hot fixes.

### Advantages

* Dev get's his branch and doesn't care about anything else.

### Disadvantages

* Extremely complicated.

## Environment branching 

### How it works?

* Environments as separate branches.

### Advantages

*

### Disadvantages

* Human brain cannot comprehend this complexity. 
* Branches are not mergeable to the master.
 
