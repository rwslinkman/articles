# Having fun with Farming Simulator and Google Cloud

Enjoying the work I do is very important to me.   
There always has to be a challenge in it, but having fun with colleagues is also a big factor.  
I love to dig deep into a subject sometimes to master it and teach other what I've learned.   
Not always though, because sometimes I really enjoy building useless software.   

At work we were recently discussing games and the possibility to play together.   
When they asked me about the games in my Steam library, I said "Well, amongst other things, I have **Farming Simulator**!"   
You should have seen the look on their faces.   
First they thought I was crazy, but to me it is a game that you can play without the pressure to "win".   
After a day of work I'm not in the mood for a shooter that gives you an adrenaline rush.   
I rather enjoy a *slow game* that really takes away stres.  
Farming Simulator is great for this and their latest edition [Farming Simulator 19](https://store.steampowered.com/app/787860/Farming_Simulator_19/) is a great piece of work!  

![Farming Simulator 2019](online-farm-data/fs19_headliner.jpg)

In the end, my colleagues got excited and also bought the game.   
We decided to rent a dedicated Farming Simulator server, so we could play together.   
The server is always online so even if the others aren't online, I can still farm the digital lands.   
It is password protected, so nobody can mess up the progress.   

Our farm currently has tractors of multiple manifacturers and we specialized in horses and forestry!    
We have a lot of fun riding the tractors and build up our farm together.   
The physics aren't super realistic, which brings a lot of fun sometimes.   
Driving a combine harvester off a cliff and then driving on like nothing is wrong, why not?

![Physics in FS19 can be really fun](online-farm-data/fs19_physics.png)
**not my screenshot*

##### Then we found something really cool
Our rented farm server comes with an administrator dashboard.   
It allows to configure some server settings, add mods to the game or restart the server.   
There is also an app for Android and iOS, which mostly has read-only functionality.   
At some point I was clicking through all the configurable options and really found a gem!

The server offers an endpoint that offers real time game statistics in XML or JSON format.   
Futhermore, there were PHP based plugins and libraries to read real time data and display it on websites.   
The API is most likely used by the mobile apps, but it was publicly accessible from the link in the admin panel.   
Clicking the link showed me XML output, but changing the URL a bit, I found that it would also return JSON!   
This is great, because it makes processing it easier.   
The only downside for me was that it does not collect statistics over time.   
It will return the current state of the server & who is online right now, but nothing to make cool dashboards out of.

### Harvesting data
So I set myself a goal. I wanted to harvest something more than digital grains to feed to my digital horses.  
Recently I have been getting more into the Serverless principle.   
For those who feel unfamiliar: the idea is that you just write the code and let your cloud provider care about hosting.   
You basically just code what it **does**, not necessarily **how** or **where**.   
A great example of this are *Cloud Functions*.   

Probably you have heard of [AWS Lambdas](https://aws.amazon.com/lambda/), [Google Cloud Functions](https://cloud.google.com/functions) or [Azure Functions](https://azure.microsoft.com/en-gb/services/functions/).   
You specify a trigger and the cloud will execute your *function* to handle it.   
Triggers could be many things, amongst others:
- HTTP requests
- A message appeard on a message queue
- Someone asked a question to your Google Home or Alexa app
- File uploads to your storage buckets 
- Database CRUD events

Whatever you can come up with, could be a trigger for your function.   
For my use case, I needed a way to build up history from a real time data API.   
Polling the API is definitely the way to go here.   
This could be perfectly set up with a cron job.   
I chose 5 minutes to be my interval and then I started thinking.   
Running an external system to send triggers to my cloud function using HTTP seemed very error prone.   
So is there a way you can run a cron job in the cloud?    
It would really offload a lot of work for me, which is nice for a basic pet project like this.   

#### Setting up the cron ticker
With most questions, I am probably not the first person to come up with this question.   
People must have wondered before how to build a *cloud cron trigger* for whatever they want to do.   
Turns out I was right, because I found [this article](https://firebase.googleblog.com/2017/03/how-to-schedule-cron-jobs-with-cloud.html) from the people at Firebase.   
They explain how to set up an [AppEngine](https://cloud.google.com/appengine/) instance that does the work for me.  
It will send a basic message to a Google PubSub topic, which will trigger my function.  
[Google PubSub](https://cloud.google.com/pubsub/) is in essence Google's message queue alternative. It has a wide range of configuration and usage options.   

AppEngine is Google's compute platform in it's most basic form.   
They provide you with a machine of your choice in the cloud, you manage it yourself.   
It is not so much in line with the Serverless principle, since the AppEngine instance *is* the server.   
Still it offers great functionality at the scale you desire.   
My use case required really nothing but a cron job, so I chose the smallest scale available.

Configuring an AppEngine is very easy, assuming that you have the `gcloud` command line tools set up properly.   
To create an `app` in Google AppEngine, you just run the following command.   
You can find my `app.yaml` configuration below that keeps the instance scale to a minimum.
It has the smallest instance type and the smallest automatic scaling set up.   
I picked NodeJS 8 as the `runtime` so the AppEngine instance will have that automatically installed.   
After booting, it will look for an `app.js` file to run.   
Once it is found, it will be executed and you can run your NodeJS app from that point on!

```
gcloud app create
```
![AppEngine configuration for the cloud cron ticker](online-farm-data/dataharvest-appengine-app.png)

Conveniently, AppEngine also has a very simple way of configurating cron jobs for AppEngine instances.   
It requires specifying a `cron.yaml` file in the project root, right next to `app.yaml`.   
The cron file is also picked up by the [Google Cloud Console](https://console.cloud.google.com) web interface.   
For multiple cron jobs, simply specify more entries to the array.   
My `cron.yaml` configuration looks like the screenshot below.   

![Cron job specification for AppEngine instances](online-farm-data/dataharvester-appengine-cron.png)

The `description` property is only used by the Console to show you what the cron job is for.   
Google defined a basic timing format, an example is shown in the `schedule` property.   
The `url` property in the cron job points to a GET endpoint on **localhost**, the AppEngine instance itself.   
Here, the NodeJS app is running. [Express.js](https://expressjs.com/) is waiting for requests to put a message on the PubSub topic.   
The code for it is quite simple and looks like this.   

![App code to put messages on PubSub topics](online-farm-data/dataharvester-publisher.png)
*Boilerplate code for setting up an Express.js app was omitted for convenience*

Google offers a very useful [NPM package](https://www.npmjs.com/package/@google-cloud/pubsub) to push messages onto the topic.  
Since I'm only interested in the event itself, it does not matter what I put in the PubSub message.   
After succesfully publishing, it logs a message to the Google Cloud and shuts down.   
So now, there is a message on the PubSub topic and I need a Cloud Function to act on it!
