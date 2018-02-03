# The Blue Alliance Live Dashboard

This is a web page to view your current teams events/matches with a live updating board of events that are occurring. If you had a small projector you could project this on a screen in your area to view how well you are doing compared to other teams.

While undergoing development, we should develop with the mindset that this could be accessed by any team at any time. But I am restricting one piece of it (account sign-up) to only be individuals I give access to since this structure is a pay per request type deal. I don't want it to get expensive during the development process until proper sponsorship or funding is in place. But this structure is meant to auto scale (described in details below).

>This project is not maintained or supported by [The Blue Alliance] and we do not have any affiliation other than using their APIs with [The Blue Alliance]. We would love their time and support once this project is fully developed to a minimum viable product.

## [AWS SAM]

[AWS SAM] stands for Amazon Web Service Serverless Application Model. This standard describes an architecture that is based around the idea that you don't have to stand up a server with code on it that is constantly listening for things to work on. Instead, it is reactive and only does things when it is told what to do.

In a normal (mostly used still) server model. A developer creates a server that listens to for requests. When a request is made, it decides what action to do then performs this action.

In a serverless model, functions are triggered from other services. When a request comes in a function is executed and returned.

Now the functions and gateways described below are all still on a server but the services we are calling are "asleep". They are not executing and don't take up CPU/Memory resources until triggered. One great thing about this approach is you only pay for what you are using instead of server time. Another pro is we don't have to worry about adding more resources or removing resources to keep performance. Since everything is triggered if it is triggered more often then the functions are just called more often and we don't have to provision more resources during peek times.

One thing to notice is even though this application may seem difficult, there is not much code to write to implement this application. A few [AWS Lambda] functions and a static page hosted on [Amazon S3]. Most of the work is in the configuring of the services but once that is done it should not be hard to maintain.

## Structure

The following is using [AWS] services and is required by this setup.

![Live Dashboard Diagram]

[The Blue Alliance] has a way to register for webhooks which are called when an event has happend (Upcoming Match, Match End, etc.). To be able to tie into this, we need an API. Since we are going for the [AWS SAM] standard, we don't have a server to put code on. That is where [AWS Lambda] and [Amazon API Gateway] come in.

[AWS Lambda] allows us to run a function (AKA lambda function) whenever it is triggered by another service. In our case, that other service is [Amazon API Gateway]. This gateway is treated like a normal JSON API where we can make GET, POST, PUT, and DELETE on. But instead of calling a server, it triggers a specific [AWS Lambda] function to be called and executed. This lambda function can do anything a normal server can do, but the idea is to keep it small, lean and fast.

In the case of [The Blue Alliance]'s webhooks we point it to the [Amazon API Gateway] which then triggers an [AWS Lambda] instance. This instance will then need to check if it cares about the event. If so, this lambda function will publish to [AWS IoT].

[AWS IoT] is a [Pub/Sub Service]. This means we can have a publisher ([AWS Lambda]) create a message (the event) on a topic that will then notify all subscribers (the client's web browser) of the message.

For the client's web browser to be able to subscribe to these topics, it first must be authenticated and authorized. [Amazon Cognito] allows for the authentication piece (making sure the user says who they are) which will then call into [Amazon IAM] to authorize (give permission) the user to do a certain action (subscribe to [AWS IoT] topic).

Another piece that [Amazon Cognito] is used for is to authorize the user to be able to hit certain endpoints on our [Amazon API Gateway]. The one for the webhook needs to be open but all the other endpoints need to be protected.

For the client's web browser we have to server up pages for them to actually view. [Amazon CloudFront] and [Amazon S3] will be used to serve up these static files (static here means the files don't change but the content on the page can). [Amazon S3] is where we will store these static files and [Amazon CloudFront] allows us to securly (SSL/TLS) serve up these webpages from [Amazon S3] using a custom domain that is registered with [Amazon Route 53].

[Amazon Route 53] is [AWS]'s DNS and Domain Registrar service. Any DNS and Registrar will do but since we are in the [AWS] stack it is easier to use it here. With [Amazon Route 53] we can point our domain to [Amazon CloudFront] and [Amazon API Gateway] appropriately for us.

There is some information that we don't get from webhooks though like team information or event information and match scheduels. So [Amazon API Gateway] will call more [AWS Lambda] functions here and populate with the proper tokens and call [The Blue Alliance] APIs then return the response back to the user. This is not ideal but unfortunately to keep a completely static frontend (webpage) we can't put secrets like that in the code.

## Implementation

To be able to allocate and configure all of these resources by hand will be cumbersome but there are tools that help us out.

[AWS CloudFormation] is way to tell [AWS] what resources we want and how we want them configured just by passing in JSON or YAML (we will be using YAML). This is great because I can distribute this file and you can stand up your own server on your own account. It also allows for the architecture to be version controlled.

[AWS SAM-Local] is an extension of [AWS CloudFormation] that allows us to generalize some of the concepts (use a more generalized function attributes instead of [AWS Lambda] attributes). It also allows us to test [AWS Lambda] functions locally on our machines (with an [Amazon API Gateway] sitting in front of it).

[AWS CodeDeploy] will be used to deploy our code where it needs to go. We will need to build our static pages (minimize CSS and JS) and deploy them to [Amazon S3] for consumption. We will also need to package our [AWS Lambda] code up and deploy them appropriately as well.

## Notes

I am still working out some details around how everything needs to be configured and setup. There are a lot of moving pieces here but through my initial research, this architecture is feasible and scalable.

One thing I am trying to decide on is how to implement CI/CD (Continuous Integration and Continuous Deployment). [AWS CodePipeline] and [AWS CodeBuild] are options here but [TravisCI] is also a good choice here as well.

Another thing that is not clear to me is how to configure [AWS Lambda] with [The Blue Alliance] token for API calls.

## TODO

All of these items I know are feasible. I have seen documentation stating how to do some of this. I just don't understand it well enough to start implementing some of these features.

* [ ] Decide how to structure this repository for multiple project like applications ([AWS Lambda] functions and static webpage)
* [ ] Decide how to properly vendor files for [AWS Lambda] code
* [ ] Decide on a CI/CD path that is most fitting for this project
* [ ] Research how to deal with configuration values for [AWS Lambda]
* [ ] Research how [Amazon API Gateway] authorizes a request using [Amazon Cognito]
* [ ] Research how [Amazon Cognito] can return tokens to use for [AWS IoT]
* [ ] Research how to tie an [Amazon Cognito] identity to a team
* [ ] Decide how to best structure topics in [AWS IoT]
* [ ] Figure out how [AWS Lambda] figures out which events to ignore

<!--Links-->
[Amazon API Gateway]: https://aws.amazon.com/api-gateway/
[Amazon CloudFront]: https://aws.amazon.com/cloudfront/
[Amazon Cognito]: https://aws.amazon.com/cognito/
[Amazon IAM]: https://aws.amazon.com/iam/
[Amazon Route 53]: https://aws.amazon.com/route53/
[Amazon S3]: https://aws.amazon.com/s3/
[AWS]: https://aws.amazon.com/
[AWS CloudFormation]: https://aws.amazon.com/cloudformation/
[AWS CodeBuild]: https://aws.amazon.com/codebuild/
[AWS CodeDeploy]: https://aws.amazon.com/codedeploy/
[AWS CodePipeline]: https://aws.amazon.com/codepipeline/
[AWS IoT]: https://aws.amazon.com/IoT/
[AWS Lambda]: https://aws.amazon.com/lambda/
[AWS SAM]: https://github.com/awslabs/serverless-application-model/blob/master/versions/2016-10-31.md
[AWS SAM-Local]: https://github.com/awslabs/aws-sam-local
[Pub/Sub Service]: https://en.wikipedia.org/wiki/Publish%E2%80%93subscribe_pattern
[The Blue Alliance]: https://www.thebluealliance.com/
[TravisCI]: https://travis-ci.org/

<!--Images-->
[Live Dashboard Diagram]: docs/TBALiveDashDiagram.png
