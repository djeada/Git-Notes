## Monorepo

Single repository for many project. Monorepos are more suitable when you are working with a monolith. 

### Advantages

* Cooperation. Because everyone is using the same repository, reusing code is much easier.
* Dependency management is simplified because each dependency change only needs to be implemented once.
* Refactoring is now easier. You may immediately apply the modifications to the whole codebase. 

### Disadvantages

* Git slows down as the codebase grows in size. 
* Feature branches are more difficult to keep track of due to their sheer number.
* Encourages tight coupling. It is simple to import from the same codebase.
* The larger the codebase, the more difficult it is to grasp. Good luck with the training the new hires.
* Because we always have to deal with the entire codebase, CI/CD setup may be complicated and the system easily overworked.
* If the master ever breaks, we're in big trouble. Instead of affecting a handful of people, a force push will effect the whole workforce. 

## Multirepo

One repository per project.

### Advantages

* Each service and library have their own versioning.
* Code check-outs and pulls are small and separate, so there are no performance issues even as the project size grows.
* Teams can work independently and do not need access to the entire codebase. 
* Faster development and flexibility for each team.

### Disadvantages

* The dependencies and libraries used across services and projects must be synchronized on a regular basis.
* Encourages a siloed culture, resulting in duplicate code.
