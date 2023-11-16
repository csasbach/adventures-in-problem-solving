# 'Stub Repo' cloud function CI/CD automation
I was working on a team that was just starting to move into the cloud.  We were working in AWS and starting to use Lambda as a way to implement simple business logic that needed to be performed either by request or on a schedule.  We were also doing so with the intent of paving the way for the rest of our team and eventually the rest of the business to follow in our footsteps.  These were small pieces of functionality but with a lot of boilerplate code to get all of the CI/CD automation set up so that one could simply commit to main to trigger a full deployment to the cloud.  The business wasn't ready to support a shared repository to host custom packages that would be easy to hand off to juniors or other less experienced teams so this also made things like code reuse for core business logic difficult.

The solution I came up with was a CI/CD pipeline to automate the creation of other CI/CD pipelines.  The basic idea is that you have a 'stub' repo that has all of the boilerplate to set up a full CI/CD pipeline for a Lambda cloud function, and it has all of the baseline general business logic that's needed. There is a 'meta' pipeline whose entire function is to create new repos and pipelines attached to those repos that have the capability to deploy to both dev and prod environments.  From there we developed command line tools so that you could simply pull down the stub repo, run a few commands, and then have a repo of your own with a fully functional CI/CD pipeline in the cloud that deployed the initial commit all the way to the dev environment.  Your next command could be to run integration tests locally or against the dev environment.  Then all you needed to do was make your first change and run the tests again.  When your work was complete you could make a pull request and when that got merged into main your code was deployed to production.

This meant that going forward nobody had to put in the repetitive work of setting up the CI/CD infrastructure.  If code needed to be re-used it was either added to the stub repo to be duplicated in all future Lambdas that used that version of the stub repo or higher.  Another way to reuse code was to fork repos when they were at a level of abstraction that was reusable.  For instance, you may have one generic baseline for all Lambdas, a fork that has boilerplate to be triggered by an API gateway, and another fork that has boilerplate to be triggered by SQS, etc.  Downstream developers simply chose the stub repo that made the most sense for their project.

## Bonus Rant
This solution also solves a major problem that I have experienced with micro-service architectures and that I think is at the core of a lot of people's criticisms of micro-service architectures.  Shared dependencies.  However, this problem can be avoided, and because I avoid it, I love using microservices.  We've all had DRY and other design principles drilled into us so much that we treat violating them as an unforgivable sin but we shouldn't do that.  We should treat *every* design choice with a healthy amount of skepticism.  The problem with excessive shared dependencies in micro-service architecture starts when you begin to place core business logic in an ever-deeper hierarchy of shared dependencies.  In the most egregious implementations, every deployed cloud function, container, virtual machine, app service, or anything that runs that core business logic takes on a shared dependency, usually many shared dependencies, shared with many other tiny pieces of infrastructure. 

If you float every dependency to the latest version, then at scale you will have deployment 'explosions' that at the very least introduce complexity and risk to even the smallest change to core libraries.  At worst you actually break your CI/CD system because it gets hung on some diamond dependency or another recursive, or difficult-to-resolve, or just plain slow-to-propagate dependency problem, and now you are stuck trying to roll back a change in the face of a system-wide CI/CD failure/slow-down.

If you pin dependencies, then at scale you have to coordinate with other teams if you want to introduce breaking changes and make sure that your deployments will align.  Teams will miss that a change is breaking, builds will fail, and deployments will have to be rolled back.  Failure to plan on one team's part will not constitute an emergency on another's.  Deadlines will be missed.

However, if you are willing to allow _some_ code to be repeated by convention, if not by hand, rather than having shared dependencies, then you can reduce the inter-dependency problem for micro-services down to those things having to do with the contracts between them, and therefore only to do with their first order neighbors (usually).  Of course, apply this advice with extreme moderation and common sense, as the rampant duplication of code without careful thought is still a very bad thing to do.